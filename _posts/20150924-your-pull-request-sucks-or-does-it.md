title: Your Pull Request Sucks (or does it?)
link: http://aimeeault.com/2015/09/24/your-pull-request-sucks-or-does-it/
author: aimeeault
description: 
post_id: 1577
created: 2015/09/24 19:43:50
created_gmt: 2015/09/24 19:43:50
comment_status: open
post_name: your-pull-request-sucks-or-does-it
status: publish
post_type: post

# Your Pull Request Sucks (or does it?)

One of the things that slows me down the most as a developer is getting roadblocked on a pull request. I can spend a frivolous amount of time, say 15 minutes, actually producing and testing my changes and then on rare occasion be stuck waiting hours or days for it to be reviewed. There's only so much you can do to alter another person's availability to review your code, but what hit me over time is that other developers are like me: they switch contexts just as much as they switch git branches. People need context! Unfortunately, reading a diff is not a fantastic way to establish context and can take just as much time as it takes to review the PR itself. And not having context can cause people to procrastinate or simply _forget_ to review your pull request until you bug them to do so. I  have alway written pull requests like any other developer might. I explained _what_ the changes were, relative to nothing else. The title of the pull request would be well-written and explain the ultimate goal of the changes made and because of that, I assumed that the changes would be implicitly understood by the reviewer. Alas, other developers do not necessarily work on the same area of the application as I do. Or have not recently worked on it. Or may misunderstand the human factor of what the changes are trying to achieve.  As someone who has been looking at the code for maybe days, I want to say, **"That's so obvious, do I really need to explain it?"** The answer to that is: **"Maybe not, but what does it hurt to do so?"** Who knows, maybe in 2 weeks, you'll need to revisit this pull request. Maybe in 2 months a bug will be discovered that is the result of your changes and other people will get involved. Either way, there is no harm in transparency. I'll use examples of pull requests I have done to break down some ideas on what makes for bad, okay, and good pull requests. I feel no shame in this because I've learned from it! I work with several people who have produced awesome pull requests that were constructed in ways I found impressive, but putting other people's work on display is weird and not cool, so this is sticking strictly to stuff I've done to highlight things. 

### What a Crappy Pull Request Looks Like

This is a pull request I did within my first month of working for Treehouse: ![Screen Shot 2015-09-24 at 2.29.28 PM](https://s3.amazonaws.com/aimeeault.com/Screen-Shot-2015-09-24-at-2.29.28-PM-1024x358.png) **Here's why it sucks:**

  1. What is "the right thing"?
  2. There's mention of BugSnag, but no link to the actual _error_ in BugSnag.
  3. "Make bugsnag stop crying" is very, very colloquial language. This is more of a nitpick on myself than anything, but if another developer were to look at this and speak English as a second language or not speak the same dialect of English as me, they might be ever-so-slightly confused by that description.
  4. No explanation of what I changed.
  5. Or why I changed it (other than to stop an error from occurring)
This pull request was almost certainly an emergency hotfix. In fact, I'm pretty sure it was me hotfixing _someone else_'s code. Which, for all practical purposes, is the ideal situation for writing a very well-formed pull request (and tagging the responsible party!). It was a one-line change that adds a presence check to a variable. But a year and a half later, I don't have any context for what this change was related to, which previous commit caused the issue I was fixing, or if there was even a GitHb issue logged for it. The pull request was reviewed and merged and probably was okay, but the title and description for it are horrid! **Reviewing a pull request should not make someone feel like they are solving a dramatic mystery.** ![tumblr_nuh1vgNaCo1qbzzgco1_1280](https://s3.amazonaws.com/aimeeault.com/tumblr_nuh1vgNaCo1qbzzgco1_1280.gif)

### What an OK Pull Request Looks Like

![Screen Shot 2015-09-24 at 2.37.33 PM](https://s3.amazonaws.com/aimeeault.com/Screen-Shot-2015-09-24-at-2.37.33-PM-1024x669.png) **Here's why it's okay (but not necessarily good):**

  * It references an issue (which is not always going to be available if you are building a feature or handling tech debt, but in this case, it's applicable)
  * It humanely explains how a model works prior to the changes to give the reviewer a comparative understanding of what changes they are about to review. After all, not every developer intimately knows all the logical constraints of every class in the application they work on.
  * It explains why there is a problem with that model's behavior both in terms of application logic and actual use case.
  * It proposes a solution in response to the problem explained.
  * It explains additional requirements related to the pull request (needing to run a job to fix data)
  * It acknowledges room for future improvements that might be outside of the scope of the pull request, in case those do come up in the discussion.
**Here's why it's not good:**

  * It doesn't provide information on how it should be tested.
  * It doesn't cover any concerns or additional implications that might be related to the changeset (in this pull request, I don't really think there were any, but I think with a larger pull request that touches more classes, this might be a problem).
  * The language used in the title is okay, but it's not completely clear. "point award" is referencing a model called "PointAward" and yet "vote" is actually referencing a model called "ForumVote." For anyone that regularly looks at this code, they will likely immediately know what I'm talking about, _but_ if someone new to the application were to look at this pull request, they might not know what a "point award" is (or if it's a model even) or maybe they will go looking for a model called Vote and be absolutely confused when they don't see anything named that.
**A pull request should direct people to the appropriate parts of the application that are touched without confusion or having to ask for more information.** ![anigif_enhanced-buzz-4693-1416329884-4](https://s3.amazonaws.com/aimeeault.com/anigif_enhanced-buzz-4693-1416329884-4.gif)

### What a Good Pull Request Looks Like

This is where things start to get more subjective. I'm using an example of a very large pull request I worked on a few months ago for this, in which I was maybe overly cautious about, but I think some the ideas surrounding it can be applicable to smaller pull requests as well. Is this pull request the best pull request in the history of the world? No, of course not. In fact, a few small-ish bugs emerged from it despite having a fair amount of clarity. The point is that it minimizes confusion about the intent of the pull request.

## Comments

**[Joel](#178 "2015-09-24 20:57:19"):** This is fairly similar to a lot of things that work I've done in consulting and internal products. There are a few things I look for in a PR and you've covered the base case very well. There are some other edge cases where things are in a state that deserves consideration. A tag in the title such as `[WIP]` or `[DND]` tells me either this may not be ready for review or the PR creator thinks it should not be deployed for some reason, which is hopefully elucidated in the PR itself. This can also be achieved easily with Github tags, though those are hard to maintain cross-repo. Sometimes PRs require data wrangling before, during, or after deploy. For those, I like to see a nice set of instructions labeled appropriately (e.g. "Post-Deploy", "Pre-Deploy", etc.). This might include things like updating data in the DB, removing/archiving something in an external system, or creating another PR to deal with leftovers from in-flight jobs. Pro-tip for formatting would be to just have a "What" and a "Why" section on every PR and pretend you're explaining it to someone who has never used the system (within reason), and you're doing very well. I think another thing worth pointing out is urgency. Some PRs are more urgent than others. In-band means of communicating that might be specific things in the name of the PR, or specific github tags, though often it is an out-of-band communication. Every team finds their own thing that works for them. Nice roundup! :)

**[Adam](#179 "2015-10-05 12:17:25"):** Hello Aimee! That's a nice post! Good pull requests are hard to find - because they are hard to write. We as engineers are more comfortable with the code so it is easy to forget these descriptive tasks. It is good to have a "suggeference" (suggestion + reference) :) In practice, I'm more used to a different approach. Much of the info you've put in the pull request goes elsewhere. For example, test steps go into the ticket - and we create a lot of them, even for features and major refactoring. What was done with the code is explained in the commits that change it. (I really like http://chris.beams.io/posts/git-commit/ for example.) It is useful to us here because there are many people working in different levels. It is good to have the tests in the ticket because it's there that the testers take a look, and the long commit messages tend to be easier to access to to programmer. Anyway, those seems to be only variants of a general need: human-readable explanations to code changes. It is good to see more people looking for that :)

**[Aimee Ault](#180 "2015-10-06 12:55:19"):** Yep, I agree a ticket needs good test cases as well. In my case, the issue/ticket itself is verified by the person who created the issue/ticket, which is almost always a different person from the person who is doing critical review of the pull request itself. I'd expect the testing conducted in the pull request to be a bit more thorough because the person doing that review has more complex technical knowledge of the app that the code resides in (and is usually testing it in a different environment from the person who tests it in the issue/ticket--which would typically happen after the code has reached production). Awesome point about git commit messages! I agree, it's always handy to be able to see changes at an atomic level and get a descriptive explanation there. The PR description itself shouldn't be a replacement for that, but I think it helps a lot as the size of the PR scales, to have that overview.

