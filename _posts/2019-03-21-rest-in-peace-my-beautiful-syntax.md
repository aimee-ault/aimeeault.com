---
layout: inner
position: left
title: 'R.I.P. My Beautiful Syntax: Considering the End Lifecycle of Successful Code'
date: 2019-03-21 09:32:30
categories: development
tags: code programming
---

# R.I.P. My Beautiful Syntax: Considering the End Lifecycle of Successful Code

As a software developer, there is rarely a moment in my day where I drop everything I am doing and exclaim, "You know, I should delete this thing I wrote 3 years ago!" I'm too swept up in other things. Personally, I'm more inclined to delete someone _else's_ code--and that's solely because my job role currently is dealing with critical bugs--but if I were doing more feature development, I would probably find myself spending more time thinking about how to write _new_ code on top of old code, rather than cleaning up existing code that is probably unrelated to what I’m working on.

In either case, there’s some chance I may refactor code, but it’s rare that on any given day that I will happen to delete a method and even more rare that I will delete an entire class or file. Typically though, if I am deleting code, it’s probably because I am refactoring it.


## Refactoring Over-Engineered Code

It's easy to refactor code with the goal of adding _new_ business scope, but to retroactively remove scope requires passive vigilance of your codebase from a broader line of sight. And when I'm already busy doing other things, passive thinking is often the first thing I let go of--because no one is going to ask me for a status update on something that doesn't have a deadline.

I've been working at Treehouse for close to five years now and one thing that I have started to notice only recently, which is not unique to my company, is that it’s common to establish with time a level of comfort and familiarity with frequently visited code, to the extent that it is easy to become blind to when aspects of that code become unnecessary with changing business scope.

As a Rails developer, for example, my eyes frequently pass over standalone microservice classes. They're all well-written classes and if I were to open one at random, say a class called `InviteToReviewAppService`, and look it over, there's a good chance I might say, "This looks legit. I mean, the name says what it does and it sounds… useful?" And then I would close it and move on with my day none the wiser that this so-called _review app_ in the name doesn’t even exist anymore… and that nothing is even calling that class. 

How would I know that though?



### An Example

"Over-engineered" may sound like a slight or negative remark, but that's not always the case. Sometimes well-written code slowly accrues the distinction of being over-engineered with time, as the codebase it was added to diverges from the functional expectations that the author had in mind when they wrote it. It just happens. And that's okay--we're developers, not fortune tellers!

I frequently work with billing library code in Treehouse's primary Rails app. if you've worked with it in any codebase, billing code--you likely already know--is typically low-churn. It doesn't change often because it is considered to be complex code that needs to remain stable in handling the key operation of managing financial transactions. I avoid touching it unless absolutely necessary in fear of introducing thrash, and when I do change it, I spend upwards of two weeks following deploying said changes drenched in a thin sheen of anxious sweat as I closely monitor our bug alerting tools.

A few years ago, Treehouse switched from one payment processor API, Braintree, to another, Stripe. After only a few months, we made the decision to switch back to Braintree. This back and forth transition was not as simple as reverting a pull request, because now we had thousands of transactions in our database linked to Stripe’s API, and our support team needed to be able to continue offering support to students who paid previously using Stripe. As a result, we had to leave our Stripe integration in place and add back our Braintree integration to live right next to it. We then had to write some higher level code which helped connect transaction records to the correct API.

We didn't have a plan in place for when to eventually sunset the deprecated Stripe integration. It was our first time deprecating a payment processor integration and our support team rightfully didn’t have an estimated timeline for when they thought they’d need to stop issuing refunds for old invoices. And as engineers, the handful of developers who worked on this were even further removed from being able to predict that. 

Before we knew it, it was almost 5 years later and I was the only engineer remaining who remembered that a Stripe integration was still there and that it was something that needed to be removed.  Functionally, this code to support using Stripe's payment processor still worked and there were still plenty of old transactions from years ago that were handled through Stripe, but realistically, no one was looking at 5-year-old transactions, much less refunding them.

But, how many times in my day do you think I'm lingering around code for connecting to a Stripe API when literally no one else is talking about it? The only way it would cross my mind to remove it is if I were actively thinking about Stripe in particular or touching code that interacts with it, neither of which are true.


## Technical Debt Should Be Roadmapped if Possible


Technical debt has a weird stigma. Outside of development, no one wants to think about it because it steals time from new feature development. Which is exactly why I want to advocate for discussing it early in a project's lifecycle, during the planning phase so that in a few months' time, it’s more of an expectation than an unwanted surprise.

You might think, "But that's crazy talk. Who the hell is talking about technical debt before it even _exists_?" And bear with me. If you talk about a problem before it even exists that likely _will exist_, it fails to become a problem or a debt because it instead becomes a part of the solution that is built. You don’t have to refer to it as technical debt. But you do have to acknowledge the questions: “Is this thing I’m building permanent? Or do we need to come back and clean it up in half a year?”


### Contrived Example

You are put to the task of writing code that dispatches notifications to your users about a promotion that your company is running. The idea is brand new and it's promoting something that you need to get exposure to for your entire user base. Your product managers are not really sure if these kinds of notifications are going to happen again in the future--they're just needed now for sending out a series of notifications over the next few weeks for this one thing in particular. There is no historical data to support how successful this feature _might_ be--you won’t know until you ship it.

Your team gets together and writes out all the specs for the project. You try to stay several feet ahead in the game by anticipating what might be missing between the lines. Maybe they'll want to send these notifications to only a subset of users one week, your most active users, for example. Or the body of the notifications will need to include Markup support when they're created. Maybe they’ll want to preview the notifications before they get sent out. Maybe they’ll want to send the notifications out staggered based on the user’s time zone. You try really hard to cover all the bases on what's needed to make the project work and what might be needed in the future, and plan to build it accordingly and pragmatically so.

**You then walk away from the meeting and you do one of two things:**

* You make a very ad-hoc system for doing exactly what was covered in the specs and nothing more, understanding that you want to get this deadline out of the way quickly.
* You make a system that is open-minded to all of the potential future use-cases, knowing it is somewhat over-engineered for the current specs, with the thought that Future You will appreciate the foresight Current You gave to the task.

There’s nothing wrong with either of these approaches--they’re just a developer’s personal sense of project management style. Personally, I’ve fit into both groups at different points in my career and I empathize strongly with both.

Both of those moves carry an internal thought process of where you thought this code was going to go in the future, but also in both cases, you had no discussion about where this code will go to _die_. Because y'know, why talk about that now before you've ever done the work?

But think about that some more. If you don't talk about it during planning... when are you going to talk about it, ever? Don't shy away from bringing that question up. 

## What if no one knows when code is meant to die?

Even if you discuss the lifecycle of your code in its planning, it's entirely possible your product team won't have answer for how long they envision a feature being necessary. If it's a gigantic project, chances are, these considerations are already being discussed somewhere higher up the chain, and that's cool. But if we're talking about something small that screams, "this code is doing something extremely particular and doesn’t have a lot of potential for reuse," and no one knows how long it will be in play, it's fair to ask to plan a follow-up discussion for the future, whether that's 2 weeks down the road or 2 months.

You may reach that follow-up meeting and the conversation is as simple as, "We're still using this for something, let's follow up again in ... 4 months" At that point in time, you've planted a seed in everyone's minds to be thinking about it passively.

## Okay that sounds really annoying. Who has time for that?

I, like nearly every other human on Earth, dislike meetings, so I definitely don't go about my day planning on having meetings about non-definitive bullet points like whether or not to delete code. No one is doing retrospective meetings for a small 1-developer pull request. If I really need a sign-off to remove code for a feature, it's also fair to just make a reminder to myself on my calendar as I work on the code to follow-up with my team as a whole at a set point in time. Maybe it becomes an agenda item for a team meeting. Maybe it's something I mention in passing in a stand-up to get clarification. Either way, I let my calendar do the hard work of remembering.

It's also been mentioned to me by others that adding event instrumentation to your code can also be a very simple mechanism for monitoring dead code. This is true in many cases, but further on, we'll talk about the concept of _dying_ code, in which you may continue to see events captured despite the fact that the code instrumenting the event is no longer aligned with your business needs. Instrumentation is a great way of monitoring potentially dead code, but you still need to monitor as a huamn regularly as well.

## But Aimee, it's not my code that's dead. It's everyone else's code. And I can't tell if it's dead because... it's not my code.

This is a real problem that exists with large codebases with many contributors. It's impossible to know what every line of code that exists is there for, even if everything is perfectly documented--and it never will be. But this is also where collaborative learning as a team is also very useful. When you have a medium-or-large-sized team, there is so much value in taking the code that you are familiar with and teaching others about that code.

Sometimes the simple act of organizing your thoughts on an area of your codebase is enough to reveal skeletons that are hidden in it. And if not, when that knowledge is shared, others might notice it as it is shared with them, and a discussion may emerge.

Personally, when someone teaches me how code that I am unfamiliar with works on a broad level, even though it doesn't make me an expert on that code, it does feel slightly easier to identify what of it is dead and what of it is thriving still. I attribute that to the fact that when I am in the process of learning, I am at a heightened sensitivity to question how or why things work the way they do.

## The biggest danger of all: Some code is aging and dying, but is not dead yet.

As a developer, I'm not out looking for dead code all the time. But I work in a bug triage environment and I often work with bugs in code that I have never looked at before. Unfortunately, bug triage in development is often like triage in an emergency room and your one primary goal is to stabilize the emergency as quickly as possible. Prior to working in a team that handles bug triaging, I would have told you that critical bugs are almost never related to dead code, but I've learned that is simply not true at all. 

When we talk about code having a lifecycle, it's important to recognize that a lifecycle is a spectrum and not typically a binary distinction of "alive" and "dead." Some code is aging and nearing death--it's code that is slowly dying and becoming less relevant. Sometimes this is because the logic that drives dying code becomes inaccurate as project stakeholders make decisions without requesting that the product be updated to reflect those decisions. I’m not saying that all bugs are due to dying code--but it certainly can be responsible for a good number of critical bugs in any given codebase.

There's a big difference between dead code and dying code. The unfortunate factor here is that dying code is often malignant and necrotizing and eager to take down healthy code that lives adjacent to it. 

Doing bug triage debugging is often a very quick (and unfortunately, headache-inducing) way of sourcing dead and dying code. As I mentioned before, I often debug code that I have no previous experience with and it’s my job to become acclimated enough with that code to pinpoint where it is failing and why. When a feature is sprawled out across your codebase in many classes, this can be challenging and hard to zoom your focus into the correct place, which is why it is essential to have the correct context for how the bug occurred so that it is reproducible before you start investigating further. Once you have consistent data to reproduce an issue, you can backtrace through the request.

### An Example

Not too long ago, I worked on a critical high-priority bug where a team member was unable to refund an invoice for a customer. Given my familiarity with billing code, my initial thoughts constellated over a spread of some of our core billing models. It’s not a crazy assumption to make to assume that refunding money goes hand-in-hand with billing.

But nothing showed up in our bug alerts and I had to take a step back and do a sanity check. If this were a bug that really had to do with refunding or billing, surely it would have been reported more than just this one time, but it hadn’t. It was a very isolated incident, so much that I didn’t even have enough data to look for patterns. So I decided to let go of my feelings of comfort and familiarity with the code and just really push through the stack trace of how this code worked with the actual invoice trying to be refunded.

This started with looking at view code visible to our support team and immediately seeing that the option to refund wasn’t even available. 

The view code, in deciding whether to display the button for refunding any given invoice, looks at the ActiveRecord object itself, in particular at a method called “refundable?”. It’s not a very complex method; all it does is check if the invoice was previously paid and if the amount paid was more than 0 cents.

With the data I had, I quickly learned that the invoice was for $0. It made sense that it couldn’t be refunded. And suddenly, the bug changed from “why can’t I refund this invoice?” to “why did this customer get invoiced for $0?”

Without getting into the nitty gritty details, my company’s codebase has its share of complexities when it comes to handling pricing on invoices. We have all sorts of business logic at stake, including plan prices, the number of users on a subscription, discounts the subscription might have, and a whole range of other things. If any one of those things changes or… dare I say… dies, any code higher up the chain that relies on it dies a painful, less eloquent death with it.

In this scenario, there was some dead code tied to how we handle pricing on plans that depended on setting up discounted tiered pricing. The feature itself was dead from a business perspective, but the code continued to live, and any user on this particular plan was getting billed at $0 per person, despite the plan pricing being a different, less-free amount.

That’s the cost of dying code. It has a smell, but you can’t always tell where it’s coming from, only that everything around it is a little less comfortable than normal. 

## Code Dies: And That’s Okay.

As developers, we don’t talk much about our code dying. The thought process behind that typically can be traced to thinking that our code is an extension of who we are professionally and a measure of our intellectual success. To some developers, writing code is an art. And that’s a perfectly valid feeling to have.
 
Imagine if every year, Van Gogh had burned his paintings because they were no longer relevant and it was time to start anew.  But, writing code is a much different type of art than producing paintings. It’s much more ephemeral and the art of it is in how much of a service it provides in the time that it is needed.

### Say it with me: 

> My code dying does not mean I did not succeed. 
> I do not have to permanently preserve my code for it to be appreciated in the time that it is useful to others.

Also keep in mind: Your code does not die when it is deleted. It is a production of an ongoing learning journey that you are partaking in as part of your career. Not to mention, if you are using an appropriate code versioning system, your code does continue to live as an archive long after it stops living in production.

The next time you sit down to write code, slow down for a minute and think about how you write the code you write. Do you ever stop and think, “Oh this is a lot like that one time that I wrote a SQL query that fetched a group of users… I could probably do something similar!” I certainly do this, at least. If you don’t do this, at least take solace in knowing that other people probably look at your code and learn things from it that they re-use when writing their own code.

We mentally recycle code we write. Code does die. But it is also recycled, upcycled, and reborn.

