title: Making the Most of Redis and Sorted Sets
link: http://aimeeault.com/2014/06/22/making-the-most-of-redis-and-sorted-sets/
author: aimeeault
description: 
post_id: 34
created: 2014/06/22 18:23:58
created_gmt: 2014/06/22 18:23:58
comment_status: open
post_name: making-the-most-of-redis-and-sorted-sets
status: publish
post_type: post

# Making the Most of Redis and Sorted Sets

I recently did quite a bit of work on a project that uses Redis for its primary method of data storage. Like many developers, I've used Redis before for simple key/value retrieval, but not for much else. What I ended up learning is that Redis works phenomenally for a few very specific functionalities, one of which is very low-cost sorting of data sets. This works really great for leaderboard-type implementations, which is what I used it for on [Treehouse](http://www.teamtreehouse.com/). Ultimately, we ended up with a starting Sorted Set including several dozens of thousands of records, based on points and badges that Treehouse students earn while learning. If we were to have implemented a leaderboard without Redis, we would have run into many other pitfalls, limitations, and shortcomings, like having to recalculate the leaderboard at periodic intervals, possibly not having the leaderboard be truly "realtime," or doing expensive lookups using Active Record--none of these scenarios are ideal. But a lot of people don't realize that Redis can operate on sorted sets or what exactly it's capable of doing, so I thought I'd do a brief walkthrough. Note that I am a developer, not a mathematician or a statistician. Although I do enjoy both of those topics a bit more than I ever thought I would, I am not formally educated in either topic, just a geek with a strong passion! Also, I use [Redis's Ruby library](http://github.com/redis/redis-rb/) for managing all of this, but I'll keep most of this walkthrough open-ended by only using non-language-specific input and commands sent directly to the Redis client, which you can find in [Redis' documentation](http://redis.io/commands). I may expound here and there with Ruby to show how you can further manipulate results, but understand that you likely can do all the same manipulation using the language of your choosing. 

# What exactly is a Sorted Set?

A sorted set is a non-repeating collection of unique Strings that is associated with a score. For example, I may have a Sorted Set with two members/records in it: one for "bob", who has a score of 30 and one for "jane", who has a score of 90. Once "jane" is added to the Sorted Set, she cannot be added again, only updated or removed--she is a unique member of the set. All write operations on the Sorted Set are performed at O(log n), which is pretty good. For the sake of this walkthrough, let's pretend we're building a scored leaderboard, keyed by username, because that's the most common use case for this data type. You might also key this on numeric user ID as well, but for the sake of making everything easier to follow, I'm using alphanumeric usernames instead in this example. 

# Basic Operations

Most of the basic add and update operations on Sorted Sets with Redis are fairly similar to other data types, with a few small caveats. 

## Adding a new entry
    
    
    127.0.0.1:6379 > ZADD "leaderboard" 30 "aimee"
    

Let's break this down. "leaderboard" is the name of the key we are storing the leaderboard on. You could call this anything you want, but if you are working within an application that uses Redis for other tasks, it's probably wise to make it semantically logical, so we're sticking with "leaderboard" for simplicity. 30 is the score that we are assigning to "aimee". If "aimee" is already included in the set, this command will update the existing entry and reinsert it based on its revised score and recalculated position within the Sorted Set. **ZADD** is great if you are adding the entry for the first time with their initial score. If you are updating an existing score, it's wiser to use **ZINCRBY** to save yourself from having to do costly lookups on your database, calculating the user's total score. The idea here is that once your Redis data store has the user's correct score, it should be in sync with your database and the score can be simply incremented for better performance. 

## Incrementing score
    
    
    127.0.0.1:6379 > ZINCRBY "leaderboard" 5 "aimee"
    

Like **ZADD**, we have almost the same parameters accepted here, except that the numeric value passed in is a value that will be added to the existing score. If "aimee" is not added yet, a new record will be added that increments from a base score of 0, making her total score 5. If "aimee" were to have an existing score of 30, she would have a score of 35 after this command finished. Both of these operations are fairly simple and shouldn't be too mind-boggling. If they are for any reason, the easiest way to understand is to open up the Redis client (redis-cli) and play around! It'll be a blast, I promise. 

## Getting a ranked list

Okay, here's where the fun starts. Having just one member in my sorted set isn't going to be that interesting, so let's add a few more people! 
    
    
    127.0.0.1:6379 > ZADD "leaderboard" 10 "homer"
    127.0.0.1:6379 > ZADD "leaderboard" 25 "marge"
    127.0.0.1:6379 > ZADD "leaderboard" 55 "bart"
    127.0.0.1:6379 > ZADD "leaderboard" 70 "lisa"
    127.0.0.1:6379 > ZADD "leaderboard" 5 "maggie"
    127.0.0.1:6379 > ZADD "leaderboard" 80 "frank grimes"
    

There's multiple ways you can sort this list. _Most_ people like to sort leaderboards by score, ranking highest first, so let's try that out first. 
    
    
    127.0.0.1:6379 > ZREVRANGE "leaderboard" 0 10
    

This asks Redis to fetch scores starting from the 0th position. The 10 denotes the number of rows we want to fetch for our leaderboard. 
    
    
    1) "frank grimes"
    2) "lisa"
    3) "bart"
    4) "marge"
    5) "homer"
    6) "aimee"
    7) "maggie"
    

Well, that's great, but it doesn't give us scores... and we kind of want that, right? Let's try this again. 
    
    
    127.0.0.1:6379 > ZREVRANGE "leaderboard" 0 10 WITHSCORES
    
    
    
    1) "frank grimes"
    2) "80"
    3) "lisa"
    4) "70"
    5) "bart"
    6) "55"
    7) "marge"
    8) "25"
    9) "homer"
    10) "10"
    11) "aimee"
    12) "5"
    13) "maggie"
    14) "5"
    

Ah, there we go. Much better. But this is kind of a useless format of results, right? To make this easier to work with, I can manipulate it into a Hash in Ruby: 
    
    
    results = Hash[*redis.zrevrange("leaderboard", 0, 10, {withscores: true})]
    

And the results are _much_ nicer to work with: 
    
    
    => {"frank grimes"=>"80",
     "lisa"=>"70",
     "bart"=>"55",
     "marge"=>"25",
     "homer"=>"10",
     "aimee"=>"5",
     "maggie"=>"5"}
    

The opposite of ZREVRANGE, if you were curious, is ZRANGE, which sorts from low to high score. All sort and lookup operations on Sorted Sets perform at O(log(n) + M) complexity -- where M is the number of records fetched, which again, is _really_ good. 

### Some other sorting methods

As of Redis 2.8.9, there are a few other ways to sort results--I haven't really found much use of them but I'm sure they're useful to some: **ZRANGEBYLEX / ZREVRANGEBYLEX** \- sorts lexically/alphabetically by member name rather than score 
    
    
    127.0.0.1:6379>  ZREVRANGEBYLEX "leaderboard" 0 10
    
     1) "aimee"
     2) "bart"
     3) "frank grimes"
     4) "homer"
     5) "lisa"
     6) "maggie"
     7) "marge"
    

**ZRANGEBYSCORE/ZREVRANGEBYSCORE** \- very similar to **ZRANGE/ZREVRANGE**, the only difference here is that the second and third parameters are no longer offset and limit but range boundaries for which scores you want to return. So, if I were to run the following command: 
    
    
    127.0.0.1:6379> ZRANGEBYSCORE "leaderboard" 10 30
    

This would return all members who scored between 10 and 30: 
    
    
     1) "aimee"
     2) "maggie"
     3) "homer"
     4) "marge"
    

There is a small caveat to this. If you use **ZREVRANGEBYSCORE** instead, you will need to swap the order of your min and max scores or else you will end up with an empty set :)

## Comments

**[Dmitry](#99 "2015-12-03 09:15:53"):** Hi, thanks for great article, I want to support filtering by country functionality in my leaderboards, this should work same way as "filter_leaderboard_by_username" described, but the challenge is that there could be near 1M users per country and near 5-10M registred, so doing redis.zinterstore("new_leaderboard", ["leaderboard", "country_set"]) could take near 5-10 seconds in this case. Have anybody faced with similar problem and found some effective solution?

