---
layout: inner
position: left
title: 'Your Pull Request Sucks (or Does It?)'
date: 2015-09-24 23:44:30
categories: development
tags: code programming
---

# Your Pull Request Sucks (or Does It?)

One of the things that slows me down the most as a developer is getting roadblocked on a pull request. I can spend a frivolous amount of time, say 15 minutes, actually producing and testing my changes and then on rare occasion be stuck waiting hours or days for it to be reviewed.

There’s only so much you can do to alter another person’s availability to review your code, but what hit me over time is that other developers are like me: they switch contexts just as much as they switch git branches. People need context! Unfortunately, reading a diff is not a fantastic way to establish context and can take just as much time as it takes to review the PR itself. And not having context can cause people to procrastinate or simply forget to review your pull request until you bug them to do so.

I have always written pull requests like any other developer might. I explained what the changes were, relative to nothing else. The title of the pull request would be well-written and explain the ultimate goal of the changes made and because of that, I assumed that the changes would be implicitly understood by the reviewer.

Alas, other developers do not necessarily work on the same area of the application as I do. Or have not recently worked on it. Or may misunderstand the human factor of what the changes are trying to achieve.  As someone who has been looking at the code for maybe days, I want to say, _“That’s so obvious, do I really need to explain it?”_

The answer to that is: _“Maybe not, but what does it hurt to do so?”_ Who knows, maybe in 2 weeks, you’ll need to revisit this pull request. Maybe in 2 months a bug will be discovered that is the result of your changes and other people will get involved. Either way, there is no harm in transparency.

I’ll use examples of pull requests I have done to break down some ideas on what makes for bad, okay, and good pull requests. I feel no shame in this because I’ve learned from it! I work with several people who have produced awesome pull requests that were constructed in ways I found impressive, but putting other people’s work on display is weird and not cool, so this is sticking strictly to stuff I’ve done to highlight things.

### What a Crappy Pull Request Looks Like

This is a pull request I did within my first month of working for Treehouse:

![](https://s3.amazonaws.com/aimeeault.com/Screen-Shot-2015-09-24-at-2.29.28-PM.png)

#### Here’s why it sucks:

* What is “the right thing”?
* There’s mention of BugSnag, but no link to the actual error in BugSnag.
* “Make bugsnag stop crying” is very, very colloquial language. This is more of a nitpick on myself than anything, but if another developer were to look at this and speak English as a second language or not speak the same dialect of English as me, they might be ever-so-slightly confused by that description.
* No explanation of what I changed.
* Or why I changed it (other than to stop an error from occurring)
* This pull request was almost certainly an emergency hotfix. In fact, I’m pretty sure it was me hotfixing _someone else‘s_ code. Which, for all practical purposes, is the ideal situation for writing a very well-formed pull request (and tagging the responsible party!). It was a one-line change that adds a presence check to a variable. But a year and a half later, I don’t have any context for what this change was related to, which previous commit caused the issue I was fixing, or if there was even a GitHb issue logged for it. The pull request was reviewed and merged and probably was okay, but the title and description for it are horrid!

**Reviewing a pull request should not make someone feel like they are solving a dramatic mystery.**

![](https://s3.amazonaws.com/aimeeault.com/tumblr_nuh1vgNaCo1qbzzgco1_1280.gif)

### What an OK Pull Request Looks Like

![](https://s3.amazonaws.com/aimeeault.com/Screen-Shot-2015-09-24-at-2.37.33-PM-1024x669.png)

#### Here’s why it’s okay (but not necessarily good):

1. It references an issue (which is not always going to be available if you are building a feature or handling tech debt, but in this case, it’s applicable)
2. It humanely explains how a model works prior to the changes to give the reviewer a comparative understanding of what changes they are about to review. After all, not every developer intimately knows all the logical constraints of every class in the application they work on.
3. It explains why there is a problem with that model’s behavior both in terms of application logic and actual use case.
4. It proposes a solution in response to the problem explained.
5. It explains additional requirements related to the pull request (needing to run a job to fix data)
6. It acknowledges room for future improvements that might be outside of the scope of the pull request, in case those do come up in the discussion.

#### Here’s why it’s not good:

1. It doesn’t provide information on how it should be tested.
2. It doesn’t cover any concerns or additional implications that might be related to the changeset (in this pull request, I don’t really think there were any, but I think with a larger pull request that touches more classes, this might be a problem).
3. The language used in the title is okay, but it’s not completely clear. “point award” is referencing a model called `PointAward` and yet “vote” is actually referencing a model called `ForumVote`. For anyone that regularly looks at this code, they will likely immediately know what I’m talking about, but if someone new to the application were to look at this pull request, they might not know what a “point award” is (or if it’s a model even) or maybe they will go looking for a model called Vote and be absolutely confused when they don’t see anything named that.

**A pull request should direct people to the appropriate parts of the application that are touched without confusion or having to ask for more information.**

![](https://s3.amazonaws.com/aimeeault.com/anigif_enhanced-buzz-4693-1416329884-4.gif)

### What a Good Pull Request Looks Like

This is where things start to get more subjective. I’m using an example of a very large pull request I worked on a few months ago for this, in which I was maybe overly cautious about, but I think some the ideas surrounding it can be applicable to smaller pull requests as well. Is this pull request the best pull request in the history of the world? No, of course not. In fact, a few small-ish bugs emerged from it despite having a fair amount of clarity. The point is that it minimizes confusion about the intent of the pull request.

![](https://s3.amazonaws.com/aimeeault.com/Screen-Shot-2015-09-24-at-2.54.49-PM-1024x231.png)
![](https://s3.amazonaws.com/aimeeault.com/Screen-Shot-2015-09-24-at-2.57.01-PM-1024x967.png)
![](https://s3.amazonaws.com/aimeeault.com/Screen-Shot-2015-09-24-at-2.57.29-PM-1024x810.png)

#### Here’s why it’s good:

1. Its title expresses an objective and explains what mechanisms will be used to achieve it without being overly technical.
2. The description is well-formatted using Markdown so that it is readable to the reviewer. Obvious? Yeah, but when you’re conveying a lot of information, what looks reasonably readable to you can so easily look like a wall of text to an outsider.
3. It explains what the benefits of the changes are in a way that makes sense both from a technical and business perspective.
4. It breaks the changes down into a changelog.
5. It explains/defines jargon that may or may not be familiar to the reviewer.
6. It raises concerns in a way that prompts for feedback rather than dictating the direction the conversation contained within the pull request will go.
7. It appropriately tags people whose work is greatly affected by these changes or who may want to weigh in on the conversation.
8. It addresses issues that are possibly not within the scope of the pull request but have been observed while working on related library code.
9. It comprehensively covers all known [edge] cases for testing the changes.

#### Ways it could be better:

1. There is a lot of acronym usage in the description of the pull request. Sometimes that’s okay when it’s a concept that is more commonly communicated as an acronym than not (API) but turning something into an acronym that is not normally communicated that way out of laziness is potentially confusing (e.g. Active Merchant as “AM”).
2. Doesn’t really dig too deep into what underlying problem is being solved (“fraud” is mentioned, but certainly could have exposed more about how that was occurring historically)
3. Although it explains potential test cases, it doesn’t explain how to test those. Testing billing changes in a non-production environment is not always obvious to everyone because a payment processor’s development sandbox is behaviorally different and typically has its own test credit card numbers that you can use for producing successful transactions and declined transactions.
4. It could have explained the technicalities of the code being changed better, but in my case, this was intentional because I knew the person reviewing it was very, very familiar with the code changed.


One other thing that I like to do inside of pull requests is to spearhead conversations by making my own inline notes on code before the reviewer has the chance to do so, pointing out any areas that I am uncertain about that I am looking for suggestions on, adding additional clarity on why a particular line was changed if I think there may be even the slightest bit of confusion. That way the conversation in the pull request is bi-directional.

I like to think of writing pull requests like hosting an out-of-town guest. We have things in common, we speak the same language, we both have the same understanding of how human life works (we both know to breathe air and walk and other human things), but they don’t know all the intricacies of my town even if it might encompass some of the same things they’re familiar with from their own town or travels elsewhere. I talk like I assume they know some things but I’m not going to just jump in and say, “How about that Timbers game last week?” because I know they’ll have no idea of what I’m talking about–they are foreign to my town and they are potentially foreign to this code as well!

It’s common as a developer to think about learning from other developers from a strictly technical perspective: “Bob has more experience with this system than I do and can share information with me about it” or “Lisa is really good with CoffeeScript and might know of something that works better here…” but on a daily basis, every time you are interacting with other developers, you are subconsciously learning something about communication by trial and error. Any time you walk into a dead end with someone through miscommunication or find that something is useful to another person, that’s noteworthy and should impact how you verbalize things in the future! Don’t make people’s brains explode.

![](https://s3.amazonaws.com/aimeeault.com/irCpBtzWVDIeI.gif)

I’d love to hear other developers’ thoughts, stories, and musings on how they set up pull requests and ways they’ve found room for improvement. Do you hate my principles? Do you love them? Do you have your own different standards? Comment below.
