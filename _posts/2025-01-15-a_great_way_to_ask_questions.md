---
layout:       post
title:        A Great Way To Ask Questions
date:         2025-05-19
---

Following the spirit of [my last post on Postgres](/what_i_wish_someone_told_me_about_postgres),
this is a guide to asking questions that I wish I had the opportunity to read earlier in my career.
I'm mostly going to be noting the habits of those to whom I've looked up to in my time as a software engineer who are _great_ at asking questions.

## Let's define a "great question" 

What makes a question "good"? Here, I'm defining a good question as one that...
* ...requires a minimum of follow-on questions before understanding what the problem actually _is_
* ...has a definitive answer (even if that answer is subjective or has doubt/uncertainty, e.g. "I don't know"!)
* ...has a clear priority
* ...can help educate others

These are all attributes that make it easier for the person answering the question to help.

The assumed context here is one over text, such as on Slack or in a GitHub Pull Request.
Each "place" is going to have some differences
(you might be a bit more fast and loose on Slack than GitHub, for example)
but the core attributes are the same.

A few things can often look like a question:
* a bug report
* a blocking issue
* a complaint

However, these are _requests_.
A good request has many of the same attributes as a good question,
and may come alongside a good question,
but they are different things.
A request is much more complex:
it requires an understanding of the particular dynamic and relationship between requester and recipient.
In fact, making a request under the guise of a question can often read poorly.

So...how do you write one?

## Where we're starting from

Let's say you're learning how to program for the first time.
Consider this problem that you're working on:

You have a string which always takes the form:
```
{"id":1234,"name":"Emmy Noether","email":"1enoether@example.com"}
```
Starting out, you may not know that that's [JSON][wiki_json].

[wiki_json]: https://en.wikipedia.org/wiki/JSON

As part of your work, you need to get the `"Emmy Noether"` bit so you shoot a DM:
> How can I get the part of a string between two of the same character?

If that's the only information the person helping you gets, they're either going to
1. (Ideally) Ask for more information, taking up more of both of y'all's time, or
2. Give a subpar answer (e.g. explaining the usage of regex matches to get substrings)

## Describe the problem you are attempting to solve

A question can be for a bunch of different reasons.
E.g. "I want to understand _foo_ because..."
* "...I need to work around something on my current task."
* "...I got paged at 3 a.m. and would rather not get paged again tonight."
* "...I just think it's cool!"

This context is _crucial_ for the person answering your question.

When you leave out the problem you are trying to solve,
the person attempting to answer it has fewer options to help.
These questions often take the form of asking for how to implement a solution you have come up with instead of asking about the root problem.[^xy]

[^xy]: This is cryptically referred to as [the XY problem][wiki_xy_problem], a term seemingly beloved by overzealous StackOverflow moderators (but sadly unrelated to the classic late-2000s ABC Family sci-fi drama <i>[Kyle XY][kyle_xy]</i>)
[wiki_xy_problem]: https://en.wikipedia.org/wiki/XY_problem
[kyle_xy]: https://en.wikipedia.org/wiki/Kyle_XY

With context, they'd be able to tell what is likely the right answer
(to use a JSON parsing library!).

## List what you've tried so far

This helps both of you in a few different ways:
* You are saved from being given advice that you already know is unhelpful
* You might get an idea for something you _haven't_ tried yet
* The person helping you
  * better understands the problem you are experiencing
  * has a better sense of your understanding of the problem,
    which will help them explain the answer 

## State your assumptions

It's possible the reason something doesn't make sense is based in falsely held beliefs.
The person helping you can much more easily steer you towards a solution if they know why you needed this guidance.

## Communicate a level of urgency

When I'm reading messages in a public channel,
it's extremely helpful to know how much of an impact my intervention might have.
It's always nice to help out (and I really quite enjoy it!),
but it helps me prioritize my work if I know how important that question is to that person.
This is especially true when I'm fielding multiple questions from several individuals.

## Ask for "the rest of the class"

In general, I would urge you to ask questions in public channels if you feel comfortable doing so.[^comfort]
Such public questions and answers are a rich resource for other folks
(who may have the very same question)
to learn.
If you've ever googled a question and found the answer on a Q&A site like StackOverflow,
you have relied on other folks asking questions in public forums.

[^comfort]:
    Sadly, this is something that may not be universal for a number of reasons.
    One might be battling perceptions (or feel like they are) in asking questions because of who they are, how they look, sound, identify, etc.
    Or you might be dealing with some immobilizing anxiety.
    No shame in any of that.
    It's something that I can't give great blanket advice on in this post.
    However,
    if you're in this situation,
    consider finding someone whom you trust who can post questions for you to take the brunt of putting themselves out there.

## Revisiting our question

Let's try to layer in what we've talked about above.

After some editing, we post the following in a Slack help channel:

> I'm currently working on a JavaScript task where I get strings that look like this:
> ```
> {"id":1234,"name":"Emmy Noether","email":"1enoether@example.com"}
> ```
> and I need to get the part `Emmy Noether` to display somewhere else.
> So far I've tried iterating through the characters of the string to flag when it has the part `"name":`
> and then take the parts after the `:`
> but it runs into issues when there is a string like
> ```
> {"id":5678,"name":"\"SPECIAL\"","email":"special@example.com"}
> ```
> I'm assuming that the parts before the `:` always are wrapped in double quotes,
> but I'm not sure what the `\` means--is it something special?
> The task is due this Friday and unfortunately I'm blocked on this.

Hopefully, whoever ends up helping you out appreciates the thought you put into asking the question.
I know I would!

---

<br>
