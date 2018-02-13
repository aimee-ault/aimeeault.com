title: On Software Development and Fuck-Ups
link: http://aimeeault.com/2016/04/14/on-software-development-and-fuck-ups/
author: aimeeault
description: 
post_id: 1781
created: 2016/04/14 21:02:42
created_gmt: 2016/04/14 21:02:42
comment_status: open
post_name: on-software-development-and-fuck-ups
status: publish
post_type: post

# On Software Development and Fuck-Ups

I posted on Twitter earlier today something from Timehop. For whatever reason, historically, around April 14th, I just seem to always fuck something up. I don't know why. There's nothing significant about that date to me, but for three years in a row, I have managed to do something every year. Possibly for more than just 3 years, but maybe then I just sheepishly hid in a corner then. I reserve the word "fuckup" for things that have painfully huge aftershocks. For perspective: **In 2014**, shortly after joining Treehouse and implementing a way for our support team to issue refunds through our admin tools, I shipped something to production that completely broke that refunding tool for invoices paid via PayPal. This was humiliating for me. I was sure everyone I worked with thought I was an idiot. **In 2015**, I upgraded a large portion of our billing library to a new version of Braintree's API and while doing so, added a client-side validation for postal codes that completely blocked out British people (fortunately we noticed this almost immediately). I felt so incredibly awkward and frustrated with myself. How dare I screw up a regular expression? Who in the history of the world has ever done that? **In 2016**, I accidentally hit backspace on a portion of a `WHERE` clause in a query and, as a result, ended up sending an e-mail to a large number of people who shouldn't have received it. This bit me hard. All it took was a single keystroke. After reviewing the query multiple times, I failed to review that one. last. time. before hitting "export to CSV." If you look at those three stories together, you might decide I'm a really shitty developer. Those are very painfully embarrassing mistakes to make, but I won't deny that I've learned something from all three of them. I'm in my 12th year of my career as a software engineer and I share these stories because it's taken me 12 years to understand that owning your fuck-ups is more valuable than trying to save face. There is something suspicious about a person who has made it over a decade without fucking up. All of the developers I respect the most have a story or ten. They laugh at them in hindsight. And so do I. I don't think, "what a dumbass." I think, "Oh man, yeah, I feel you." I thought for a very long time that if I could stealthily fix a problem without outing myself, that I should. I feared the shame of people losing trust in me to be able to do something correctly. I feared that they would talk about me behind my back. That they'd be angry with me. I'm a very sensitive person and very self-critical already, so making mistakes can take a huge personal toll on me, so much that I sometimes will cry because I'm so embarrassed. I can't say I don't cry when I screw up, but I've since learned that people will trust you more if you are open and honest about your mistakes, especially when you are earnest and heart-felt in your mission to remedy what you've done wrong. Would it be nice if you hadn't made a mistake? Sure, but good luck with that. So, I've recently resolved myself to a few different things. When I know I've screwed something up, I: 

    * Immediately notify other people that I feel should know.
    * I take a step away for a moment and clear my head. If I'm distracted, I'm not going to be able to fix the problem.
    * I make an action plan. I think it's easy to panic here and try to fix something super quickly and instead stir more trouble into the mess.
    * I document the problem. Very thoroughly. I try to use laymen's terms so that anyone can understand what has gone wrong.
I reflect back on my mistake. It's _mine_. In a year's time, it's going to be a funny story. Right now, it's a moment to learn how I can prevent it. Value your fuck ups. They make you who you are.

## Comments

**[Hafiz Abdur Rehman](#1020 "2016-10-15 07:10:59"):** Wonderful. Thanks for sharing the perspective.

