---
layout: inner
position: left
title: 'You Suck at Reviewing Code (or Do You?)'
date: 2015-11-05 23:44:30
categories: development
tags: code programming
---

# You Suck At Reviewing Code (or Do You?)

Last month, I wrote a piece discussing the characteristics of poorly written and well written pull requests. And with that, I started to ponder the other side of the fence: How does one review code well?

Reviewing code can be frustrating, even when the pull request is well-written. Say you’ve been working on the same thing for 3 days now and suddenly you’re asked to review a pull request with a 35-file changeset which includes database migrations and makes significant architectural changes to the application. Big changeset or not, many PRs are very capable of taking down a site on deploy and it’s scary to find yourself responsible for that if you miss something.

### You don’t want to take the site down

I’ve done that. If you’ve spent any time working as a professional developer, you probably have too. One time, shortly after I had joined Treehouse, I spent a day or two writing some new stats collection methods that were run as part of a daily rake task. The pull request was approved, my tests passed, the  build passed in our continuous integration service. And then I deployed it. And it took down Treehouse immediately. The culprit? Bad data.

Taking a site down means doing a post-mortem and trying to figure out, “What can we do to make sure this doesn’t happen again?” And that’s great. But it’d be greater if you didn’t land there in the first place.

![](https://s3.amazonaws.com/aimeeault.com/FldBatL.gif)

### Ask questions, even if you suspect they sound dumb

One thing that I heavily advocate in reviewing a pull request is letting go of your assumptions about how things work and approaching the code as if you had no awareness of the application’s nuances. As you come across these nuances, first run an internal monologue and explain to yourself how those things work. If you can’t find a way to explain it to yourself in clear, easy to understand language, it’s fair game for leaving a comment on the merit that other developers who aren’t as experienced with the application won’t intuitively understand it.

This might sound like it’s time-consuming, but once you’ve adapted this philosophy for a brief time, it’ll become quick to you, much like speaking a foreign language is with practice.

![](https://s3.amazonaws.com/aimeeault.com/CO2oB11.gif)

### But don’t be afraid to approve.

You don’t have to be a pedantic asshole while reviewing code. I know, you have this voice in your subconscious that worries, “Will they even believe I reviewed their code if I just give it the old thumbs up without any additional comments?” Sometimes someone will really just knock it out of the park, though. It happens. You don’t have to be pedantic for the sake of fostering credibility. I swear.

![](https://s3.amazonaws.com/aimeeault.com/anigif_enhanced-buzz-18929-1420303977-4.gif)

### “It’ll only be like this for a little while… I promise…”

I’m guilty of this occasionally. It happens a lot when you work on large, moving-target type projects. There’s a compromise between understanding deadlines and understanding that you’re introducing future technical debt to an application. If you work on public-facing products, this can happen a lot.

When you see something in a pull request that looks sketchy, hackish, or like a shortcut, and the author of the code is excusing it with time constraints due to a deadline, it’s appropriate to question the timeline for remedying the workaround or at least prompting a discussion of: what the long-term plan for that workaround is, and making sure that it is documented somewhere so that it doesn’t get left behind as painful legacy code to clean up for some new developer 3 years down the road. If you ever see `#TODO: Fix this` in a pull request, a discussion should be happening surrounding that.

Mind you, this conversation should not be done in a finger-pointing manner. It should be done constructively, with the primary concern being the health of the codebase you’re maintaining. But also empathetically, with the understanding that one day, you too will be on the opposite side of the table.

![](https://s3.amazonaws.com/aimeeault.com/anigif_enhanced-1848-1411417802-5.gif)

### Share knowledge if you have it

Reviewing a pull request is not just a matter of “does this work?” It’s a matter of “does this work in the most effective and efficient way?” If you know of a way to more cleanly write something and you fail to point it out to the author, you’re doing a disservice to them, intellectually, and future code that they write. This happens a lot when working with Rails apps, because there are so many useful things available through ActiveSupport that even the most seasoned developers might not know about.

![](https://s3.amazonaws.com/aimeeault.com/anigif_enhanced-3376-1414921505-1.gif)

### It works, but does it work well?

Just because you tested something in your local development environment and confirmed that it works doesn’t necessarily mean it’s good to ship into a production environment. Performance should be a concern as well. Can you benchmark changes made? Do you have access to New Relic or other tools locally? If you suspect that there may be potential performance issues with a pull request, you should be checking these things.

![](https://s3.amazonaws.com/aimeeault.com/anigif_enhanced-5488-1416337355-5.gif)

### Does it scale?

What works now may not work well forever or even next month. Don’t think of the pull request in isolation. Think about how it will gel and coalesce with other parts of your codebase. If you have 500 users today, how do you think this code will fare when you have 5,000 or 50,000? Are there logical constraints to the code being committed that will make working in future product features difficult or annoyingly complex?  Does this addition to your codebase put it one step closer to being a monolithic piece of crap? Would it serve you better as a micro service?

Having these conversations sooner than later will save you headaches in the future. You don’t have to over-engineer. But you do have to at least consider the implications of not over-engineering.

![](https://s3.amazonaws.com/aimeeault.com/anigif_enhanced-5723-1415662903-7.gif)

### Is there something else that will do this better?

This is awkward. Someone has just written a large pull request but you know of a very simple gem or library that does exactly what they just labored over, only the gem/library you know of does it… better or more thoroughly.  Do you say something? You should. You should also leave that decision in their hands of what they want to do with that information, awkward as it is.

### Remember, your coworkers are humans too.

When I review a large pull request, I compliment just as much as a I critique. Even if it’s just a “thank you for doing this.” People need positive feedback. They need to be reminded every now and then that there’s a reason they’re doing what they’re doing. As cold and robotic as code can be, it’s on us as developers to high five and shower each other with animated GIFs and creepy smiling cat face emojis.

Remember, one day, you’ll be having the worst day in the world and someone will comment on a line of your PR with, “Woah that’s an awesome idea, :clap:” and you’ll swell with pride and happiness. Remember that every time you say something nice, someone else might be living that same moment.

![](https://s3.amazonaws.com/aimeeault.com/tumblr_ntlritqDmM1sn75h6o1_400.gif)
