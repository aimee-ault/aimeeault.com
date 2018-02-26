---
layout: inner
position: left
title: '30-Minutes-a-Day Development'
date: 2016-03-06 23:44:30
categories: development
tags: development code
---

# 30-Minutes-a-Day Development or How Not to Burn Out on Your Projects

One thing that destroys me as a developer on single-person projects is over-enthusiasm. That is, I have an idea, I get very excited about it and the following things happen:

* I dedicate a night or a weekend to executing on it.
* In the course of that time, I finish roughly 50-75% of the work on it.
* The next day or next week comes and I don’t have time to finish it because my schedule is full of other things.
* I burn out and abandon it.

This all changed when I first decided to start my side business, Creepy Postcards. If it hadn’t, I wouldn’t be calling Creepy Postcards my side business.

Creepy Postcards is a largely offline business. Customers fill out a brief form giving me information about a friend or family member, I exercise my creative writing prowess, we converse a little bit more, and I ultimately mail a postcard through the United States Postal Service.

The last thing I want as a business owner is to waste an inordinate amount of time working on a web application. That’s why I took to single-owner project planning.

As a developer, it’s easy to associate project planning with bureaucracy. It’s something I experience in my day job and although I know it has some value, I still loathe the downtime, the calls, the slow pace, and the feeling of a million sighs passing through me as I think, “I could be using this time to be productive.”

It’s especially hard to see the value of it in my side project, but that’s where it’s more valuable. In my one-person business, I don’t have product managers, data scientists, or designers. I need to list out what features I want to add _now_.

### The 30-Minute Task

In order to be quick-moving on a project, I need to break features down into short tasks, tasks that I can resolve in 30 minutes or less. Tasks that keep me actively working on the project, but allow me to walk away from it after that time has passed. This is similar to the Pomodoro technique, which breaks down work into 25 minute intervals, but ever-slightly different.

Why time and not points or some other metric for measuring workload? I hate project management tools like Pivotal Tracker. I hate the concept of something relative like “points” driving the meaning of a feature, for a couple of reasons:

* As one of my coworkers, Ryan Manwiller, pointed out once: 1 point to you does not necessarily mean the same thing as 1 point to another person.
* There becomes a problem with “inflation” of points where the value of 1 point cannot be broken down any further. You can have 2 tasks, each worth 1 point, and one of those tasks could take 15 minutes while the other could take 3 hours. If someone is dependent or blocked by one of those tasks, “1 point” does not communicate to them when they can expect to become unblocked by your work.
* When a task hits an unexpected bump, which is fairly typical in development, it is harder to account for that in a point-based system. Say I am refactoring a class and have allotted 3 points to that task, only to discover halfway through my refactoring that there is some gnarly coupling between this class and another class that is going to take a couple of days to resolve. What am I supposed to do there? Do I update my points? And if so, what metric am I basing the point value on now? This gets confusing.

That’s why I think point-based project planning is ineffective. And not just for developers. It lacks a human element to it. It has its heart in the right place, but it just does not work often enough.

Instead, I find value in writing down feature ideas, in language that is readable to any human being, and then breaking that idea down, as I said before, into 30 minute tasks. The tasks do not have to be accessible to a layperson. They do not need to be customer-focused. They can be as technical as I want, as long as they accomplish something.

The value here is that if my assessment is off on one item, I generally have a really strong idea of how skewed my timeline is because each task represents the same amount of time. But, more importantly, I break things down into such atomic pieces, that I generally can foresee issues before I hit them and can plan around that.

### Real Example of a Start-to-Finish Project Feature Planned in 30 Minute Blocks

Let’s use a concrete example. Last year, a friend made a really good suggestion to me: Offer people the opportunity to type the content of their own postcards and automate the process more at a discounted price, leaving the possibility for a full custom order.

#### This sounds like a simple idea but there are a lot of moving parts to it:

* My ordering system was not advanced enough to build on top of it. It was simply a call to Stripe’s payment processor from a controller with a hardcoded price.
* Users had no awareness of what cards were in my inventory and because of that, they didn’t have a true sense of how much content could fit on a card.
* My order form was a pain point in itself. It was cutely designed to fit entirely on top of an image of a postcard. Any additional complexity there would destroy the simplicity of that UX. And I’m not a designer so it’s hard to gauge how to handle that!


#### To get around this, I needed to do the following:

* Inventory my entire postcard stock.
* Implement some sort of management for that in my app using Carrierwave and AWS S3.
* Refactor my billing “system” into a true library that was capable of processing differently priced orders.
* Re-envision my order form, possibly being a bit more crafty with how I build orders through calls to action.


#### Piece-by-piece, these are huge undertakings, so let’s break this down some:

* Inventory my entire postcard stock:
  * Take front and back photos of each card (about 100 in stock at the moment).
  * Clean up photos on computer.
  * Store photos in S3 bucket.
  * Build a model in my app to represent cards in inventory.
  * Populate my production database with each card.
* Implement some sort of management for that in my app using Carrierwave and AWS S3.
  * Add Carrierwave and S3 gems to Gemfile.
  * Update my dotenv file and export ENV to Heroku with my S3 credentials (public access key, secret, and bucket name).
  * Update my Carrierwave initializer to use S3.
  * Build a new controller for card inventory items that includes an action for adding new cards.
  * Build out views for uploading and building new cards.
* Refactor my billing “system” into a true library that was capable of processing differently priced orders.
  * Extract billing logic out of my CardRequestsController and move any Stripe API interaction into its own class in /lib.
  * Evaluate what is part of the card request creation process and establish a service class that will create a billing task and then handle follow-up record processing.
  * Break my `CardRequest` model up a bit further to store price. In my case, a `CardRequest` is not unlike an `Invoice`–it maintains record of the Stripe transaction and is a state machine.
* Re-envision my order form, possibly being a bit more crafty with how I build orders through calls to action.
  * Establish an “available cards” gallery
  * Add “order this card!” buttons to each gallery item that pre-populate the order form.
  * Make order customizable or specific.

That’s a lot of items, but each one takes about 30 minutes (or less) total. The important part being that I feel confident that I can sit down and completely finish the item in one easy session. This means if I know I have a difficult problem ahead, I can continue to ponder on that some and keep the project fresh in my mind since I am actively contributing to it on a daily basis.

Doing a breakdown here, you can easily catch your trouble spots. If you can not break a task down into small atomic pieces that are not vague, there is a chance you are hitting a paint point in how you understand your work.

In my case, my whole section of _“Re-envision my order form”_ is vague and hazy and doesn’t have truly actionable items. I know these things will take me more than 30 minutes since I don’t have good UX experience. That’s why these things are at the end of my list, so I can mull over them some more and revise my actionable items before I get to them.

### Using This Practice for Your Day Job

30-minutes-a-day is the bare minimum. You could apply this practice to your day job too and fill 5 hours of development with 15 unique 30-minute tasks. Given how many distractions most developers face day-to-day from external forces (critical issues, unplanned calls, drop-in discussions with other developers about things you’re not directly involved with), having small atomic tasks works really well since there’s a good chance you will find uninterrupted 30 minute windows of time to do your work. You might feel really good if you knock out about 10 of the things on your list.

### Deadlines

In the case of Creepy Postcards, I’m not hard deadlined. But for you, you might have greater external forces driving your deadlines. Deadlines should be a compromise of what’s needed and what is actually reasonable. When you understand what you’re working on at the smallest unit of time possible, you should have a better understanding, ultimately, of how much time overall you will need to dedicate to your project. If you’re smart, you’ll approach this conservatively and buffer your estimate some to cover the unthinkable (development problems) or the unforeseeable (illness).

If used in a multi-developer project, it’s a must for all developers working to follow the same practice if you expect to understand your time estimate.

If external forces insist that the deadline must come sooner than what you know to be your reasonable timeline, you know at this point that compromise is needed, whether that’s additional resources (other developers) or removing scope from your project to fit within the timeline.

### Skeptical?

I understand. It sounds too perfect, right? Try it for a day though. Really, just a day. That’s all the time it takes to wrap your head around whether it will work for you. Start the day by breaking your work down into 30-minute tasks. Write those tasks down. And do those things. Even if you think, “that sounds like nothing…”–write it down. At the end of the day, you will have a much better understanding of how much work you have actually accomplished that day.
