---
layout: inner
position: left
title: 'How to Use Searchkick and ElasticSearch in Your Rails App'
date: 2016-02-05 23:44:30
categories: development
tags: code ruby rails elasticsearch
---

# How to Use Searchkick and ElasticSearch in Your Rails App For Complex Search Indexing

For reasons that elude me, I have always been obsessed with "speedcoding." That is, I like to see how fast I can implement a very large feature in a ridiculously short amount of time. I won't lie: this kind of trait goes hand-in-hand with phrases like "cowboy coding" and "Balmer peak," and with age, I've largely outgrown it, but the mood still hits me every now and then. I recently enjoyed one of these moments while toying around with some code for [Treehouse](http://teamtreehouse.com), for the fun of it.

In a past life, I spent about a year working on a team for [DeviantArt](http://www.deviantart.com) whose sole purpose was to improve search results on the site. If you were not aware, DeviantArt's search is done entirely in-house by people who have PhDs in math. They're brilliant people who will talk your ears off about facets, scoring, histograms, and tagging metadata. I didn't work on any of the search indexing services myself (which were all written in C++), but I was heavily exposed to the bits of it that were included in the main app, written in PHP. And as a result of that, I know more about search indexing than I'd like to say I know.

### CloudSearch or Ransack?

So, seeing poorly implemented search indexing tools also causes me mental anguish. That's how I feel about [Amazon CloudSearch](https://aws.amazon.com/cloudsearch/), in general, which is a tool Treehouse has used for one of its most major site features. It's not the worst indexing tool in the world but I really dislike that you have to hit your indices through an API since it's all hosted externally, which seems unnecessary, unlike hitting a CDN for assets. That's like hosting your Redis stores on a third-party service: why. But, worse, for a few internal tools, Treehouse uses a Rails gem called _[Ransack](https://github.com/activerecord-hackery/ransack)._ I don't _mind_ Ransack. It has its heart in the right place, but it leverages ActiveRecord for its querying and so, you'll have to put a _lot_ of effort into optimizing indices on your database tables if you expect it to work even halfway decently as the size of the table you're querying grows. This also assumes you're not doing expensive table joins as part of your query.

### Searchkick to the Rescue

I recently decided to see what would happen if I used ElasticSearch for a fairly complex search, using a gem called [Searchkick](https://github.com/ankane/searchkick). I really like Searchkick because out-of-the-box, there's no configuration needed. If you want to index your User model, it's as simple as adding the _searchkick_ gem to your Gemfile and a directive to the class:

```ruby
class User < ActiveRecord::Base
  searchkick
end
```

And then reindexing the model:

```ruby
pry(main)> User.reindex
```

The cool thing here is that you can also do your reindexing asynchronously so that it won't block processes or have a dramatic impact on your application's performance. You can then search all attributes of User (i.e. name, e-mail, city) like so:

```ruby
pry(main)> User.search("bob")
```

And you'll find plenty of demos and tutorials for that all over the place, but who ever needs the simplest use case?

#### I had a few different needs:

* Because Searchkick uses ElasticSearch, you can't chain scopes off of the model prior to running the search like so:

```ruby
pry(main) > User.active.search("bob")
```

I mean, you can, but it'll ignore the named scope and still run your search against _all_ User records. So I needed to be able to account for different scopes, and the above piece of code simply does not work and cannot be made to work.

* I needed to be able to sort my results by a variety of things which weren't necessarily attributes on the model itself. For example: I needed to sort by the time difference between the model's created_at attribute and its updated_at attribute. Or I needed to sort by the created_at timestamp on a child association. ElasticSearch's DSL supports a sort order constraint, but how do you sort by a value that isn't indexed with the model?
* As I started to index more things, I noticed my controller logic was growing wily. I needed some sort of presenter type class or simply a PORO to organize my Searchkick search. So I'm going to walk through how I developed this search feature, stopping to explain my thought process along the way. Because I didn't feel like getting sued by my employer for any potential intellectual property theft, I've used a completely different search feature that has absolutely nothing to do with what I was originally building a search for. Once I was finished writing this code, I was able to roll out a second sortable, filterable, search tool for another model in about 20 minutes reusing the same pattern.

### Where I Started

I wanted to index a model called Movie. Each Movie is directed by a Director and has many Actor records through a relational model called ActorRole. Searchkick allows you to override which columns are used in searches via a method called search_data.

My first step is to find out what things I need to index here!
```ruby
class Movie < ActiveRecord::Base
  searchkick
  belongs_to :director
  has_many :actor_roles
  has_many :actors, through: :actor_roles

  def search_data
    attrs = attributes.dup
    relational = {
      director_name: director.name,
      actor_names: actors.map(&:name)
    }
    attrs.merge! relational
  end
end
```

What this does is allow me to continue indexing the attributes on the model itself, but also includes a couple of other pieces of denormalized data from associations, namely the name of the Director who directed the Movie and the names of anyone who acted in the movie, both pieces of data that are _not_ stored on the Movie record. So, assuming that I have a Movie with the title "[The Room](https://www.youtube.com/watch?v=aYRydundnt8)" directed by esteemed actor, director, and writer Tommy Wiseau, I can now search for "Tommy Wiseau" and "The Room" will be one of my search results--both because he acted in the movie and directed it.

If you're used to working with relational databases, seeing denormalized data stored this way might bother you, but it shouldn't. Remember, the purpose of these indices is for aiding in searching, not for data management. Your indices do not need to look pretty--they need to simply be a collection of values that you search with, mapped to their respective data types. That's _why_ it's a separate data store from your primary database, afterall.

**You should always reindex after making changes to the search attributes, so that ElasticSearch can pick up anything new.**

### Implementing This In A Controller

```ruby
class MoviesController < ApplicationController

  def index
    query = params[:q].presence || "*"
    @movies = Movie.search(query, { page: params[:page], per_page: 20 })
  end
end
```

As you can see, the search method, provided by Searchkick, takes 2 parameters. The first is a query string. The second is an options hash. Already, I'm passing two options to set up pagination support. You might imagine how hairy this will start to get once I need to do more complex search functionality.

**I'd like to move this logic into its own service object for a couple of reasons:**

1. I'm a big fan of keeping controller actions skinny (as most Sandi Metz fans are)
2. If I later decide to add additional searches, I am likely going to reuse this logic. So let's do that.

```ruby
class MovieSearch
  attr_reader :query, :options

  def initialize(query:nil, options: {})
    @query = query.presence || "*"
    @options = options
  end

  def search
    Movie.search(query, { page: options[:page], per_page: 20 } )
  end
end
```

```ruby
class MoviesController < ApplicationController
  def index
    @movies = MovieSearch.new(query: params[:q], options: search_params).search
  end

  def search_params
    params.permit :page, :sort_attribute, :sort_order
  end
end
```

I still don't like this though. It's not generic enough, and we'll see why in a minute as we continue to build it out. I prefer to get something working before trying to refactor it.

### Extra Beef - Filtering

I don't know about you, but I've never been to a movie site that didn't offer browsing by genre. That's just an obvious thing about movies, yeah? So we need to work that into our search somehow. The problem is, since we're using ElasticSearch, we can't do any initial filtering through ActiveRecord scoping. Everything needs to take place within the Searchkick options. So we need to consider that in our search class.

```ruby
class MovieSearch
  ...
  def search
    Movie.search(query, { page: options[:page], per_page: 20, where: { genre: options[:genre] } } )
  end
end
```

The options hash is now starting to get polluted and messy, which means it's probably time to extract parts of it out into its own method:

```ruby
class MovieSearch
  ...
  PER_PAGE = 20

  def search
    constraints = {
      page: options[:page],
      per_page: PER_PAGE
    }

    constraints[:where] = where

    Movie.search(query, constraints)
  end

  def where
    if options[:genre].present?
      { genre: options[:genre] }
    else
      {}
    end
  end
end
```

These are simple use cases. But what if you wanted to filter on something a bit less obvious that doesn't necessarily seem like it would be a search keyword--like say, filtering Movie based on whether the director is still alive or has died before a certain date. Sure, that's not a common thing to filter on, but a majority of our lives as developers are building out logic that has some special meaning to our product or customer, otherwise we'd all be using pre-existing open source software and calling it a day.

To do this, I need to add that denormalized data to the search index.

```ruby
class Movie < ActiveRecord::Base
  ...

  def search_data attrs = attributes.dup
    relational = {
      director_name: director.name,
      actor_names: actors.map(&:name)
    }

    if director.death_date.present?
      relational[:director_death_date] = director.death_date
    end
    attrs.merge! relational
  end
end
```

When you reindex your model, Searchkick will pick up that you're indexing a date and you'll be able to evaluate it as such in your search:

```ruby
class MovieSearch
  ...

  def where
    where = {}
    if options[:genre].present?
      where[:genre] = options[:genre]
    end

    if options[:director_deathdate].present?
      where[:director_death_date] = { lte: options[:director_deathdate] }
    end

    where
  end
end
```

`lte` and `gte` are both parts of the ElasticSearch DSL, if you were curious. And honestly, I think it'd be weird if someone had a future death date listed but, again, we're just toying with data here 😉 You could even filter within a range if you had two date objects, using both `lte` and `gte`.

I could continue adding ways to filter on additional attributes, but it's largely rinse and repeat from here, with some mild Ruby refactoring along the way.

### Sorting

This was the one part that gave me pause, but it's not wildly different from filtering via `where`. Say I present Movie results in a table that contains columns for its title, genre, release year, and director's birth year. The first three items are simple to sort on because they're already indexed attributes. Just like we had to add the director's death date for filtering on that, we'll need to add the director's birth year in order to provide sorting options for that:


```ruby
class Movie < ActiveRecord::Base
  ...

  def search_data
    attrs = attributes.dup relational = {
      director_name: director.name,
      director_birth_year: director.birthdate.year,
      actor_names: actors.map(&:name)
    }

    ...

    attrs.merge! relational
    attrs
  end
end
```

And add a sorting option to our `search` call:

```ruby
class MovieSearch
  ...

  def search
    constraints = {
      page: options[:page],
      per_page: PER_PAGE
    }

    constraints[:where] = where
    constraints[:order] = order

    Movie.search(query, options)
  end

  def order
    if options[:sort_attribute].present?
      order = options[:sort_order].presence || :asc
      { options[:sort_attribute] => order }
    else
      { }
    end
  end

  ...
end
```

Pretty simple. One thing I want to point out here is that ElasticSearch allows you to sort by `_score`. You can sort by multiple attributes, so you might want to consider continuing to sort by score, because that's one of the niftier things about ElasticSearch--as it receives more queries and its indices grow, it grows more intelligent about which results are relevant, and sorting by score will weight the more relevant results towards the top.

Moving on! Now, our `MovieSearch` class, in full, looks like this:

```ruby
class MovieSearch
  PER_PAGE = 20

  attr_reader :query, :options


  def initialize(query:nil, options: {})
    @query = query.presence || "*"
    @options = options
  end

  def search
    constraints = {
      page: options[:page],
      per_page: PER_PAGE
    }

    constraints[:where] = where
    constraints[:order] = order

    Movie.search(query, options)
  end

  def where
    where = {}

    if options[:genre].present?
      where[:genre] = options[:genre]
    end

    if options[:director_deathdate].present?
      where[:director_death_date] = { lte: options[:director_deathdate] }
    end

    where
  end

  def order
    if options[:sort_attribute].present?
      order = options[:sort_order].presence || :asc
      { options[:sort_attribute] => order }
    else
      { }
    end
  end
end
```

### Refactoring

And this is where we can start to think about refactoring potential. Some of the logic in this class is tightly coupled to the Movie class, but some of it is generic enough that it could be used for any model using Searchkick for searching. So maybe it's time that we break this into a base class which our MovieSearch class can inherit from:


```ruby
class Search
  attr_reader :query, :options

  def initialize(query:nil, options: {})
    @query = query.presence || "*"
    @options = options
  end

  def search
    constraints = {
      page: options[:page],
      per_page: options[:per_page]
    }

    constraints[:where] = where
    constraints[:order] = order

    search_class.search(query, options)
  end

  private def search_class
    raise NotImplementedError
  end

  private def where
    {}
  end

  private def order
    if options[:sort_attribute].present?
      order = options[:sort_order].presence || :asc
      { options[:sort_attribute] => order }
    else
      {}
    end
  end
end

class MovieSearch < Search
  private def search_class
    Movie
  end

  private def where
    where = {}

    if options[:genre].present?
      where[:genre] = options[:genre]
    end

    if options[:director_deathdate].present?
      where[:director_death_date] = { lte: options[:director_deathdate] }
    end

    where
  end
end
```

I added a new method called `search_class` that raises a "Not Implemented" error on the base class. If the child class fails to implement that, as it should since it specifies which model to search, that error will be raised. Because we were able to extract so much into the base class, the `MovieSearch` only had to include the `search_class` method and a `where` method for movie-specific filtering logic. You could potentially override the `order` method as well if you wanted to have a default sort order, like say, if you were filtering by the director's death date, you always wanted to make sure you sorted results by that attribute in descending order.

### Final Thoughts

#### Cache-busting

There's a lot more you could do here. I just wanted to dig in a little bit beneath the surface of all the tutorials that choose to cover the most basic use case. One thing, I'd like to point out though is that when you use ElasticSearch on a model that has child associations in its search data, you'll need to make sure that the parent model gets reindexed when the child object gets altered. Searchkick automatically reindexes when the model itself changes, so just as you would handle cache-busting, you need to make sure your associations have a `touch: true` directive to trigger this reindexing!

A former coworker of mine also shared with me an interesting, alternative approach to handling these kinds of issues by [adding instrumentation to the child model's lifecycle that notifies](http://api.rubyonrails.org/classes/ActiveSupport/Notifications.html) when it needs to be reindexed and then subscribing to those notifications to ensure that a reindex takes place. I really like this approach too and would heavily advocate it if your searches start to get heavily burdened by ActiveRecord associations.

#### "I hate service objects and think aspect-oriented programming is the way of the future!"

That's fine and I understand there's a couple of different ways you could propose to architect the code I wrote here. Instead of building a service object, you could make a Searchable aspects module that is included in your model and override the {% ihighlight ruby %}where{% endihighlight %} and {% ihighlight ruby %}order{% endihighlight %} methods within your model instead of building separate search classes. On one hand, I'm not a gigantic fan of this because it places logic that doesn't belong on the model in the model. On the other hand, if you look at how `Searchkick` works, it's already doing this by forcing you to override its `search_data` method on the model. In short: ¯\_(ツ)_/¯

#### Code Climate and Flog

True story: Flog will hate you if you have even the slightest bit of complexity in your `search_data` methods and if you choose to extract out relational data into their own methods in cases where you were needing to pass blocks to `map`, you'll start to get that stinky feeling that you're violating the [Law of Demeter](http://c2.com/cgi/wiki/LawOfDemeter?LawOfDemeter). So, in a way, I feel like having `search_data` on the model is a code smell that Searchkick forces you to commit, but, to reiterate: ¯\_(ツ)_/¯

#### "I love you for writing this but I know something you don't know and want to contribute!" /
"I love you for writing this but I'm a hands-on learner and want to download the code to play with myself!"

Did you find this article useful but want to have a more hands-on learning experience? Good news! I've [put this code on Github](https://github.com/aimee-ault/SearchkickSearch) where you can clone it and play with it on your own.
