---
title: "Why Use a Board?" 
author: "Ian Firkin"
date: 2020-11-24 
draft: false
---

A lot of teams have moved from being colocated to distributed recently,
and quite a few people are jumping in to defend their tool of choice for
representing "cards on a wall." Jira-bashing is always popular, but one
thing I've noticed is that folks aren't necessarily optimizing for the
same things; the result can often be a focus on tooling, and reducing
the discussion to "which board is better", rather than understanding
what problems these tools help us to solve.

This isn't limited to "online" boards or electronic tools--I've seen
many teams miss out on the most valuable parts of using a board, whether
they're distributed, or collocated with actual notes stuck to a wall. By
talking about some of the principles that guide my own choices, I hope
to show a different perspective on what makes a board valuable, and to
help other teams get the most out of whatever tool they're using--even
Jira!

Boards Provide Feedback for Improvement
=======================================

Whether we're talking colocated or distributed teams, the use of
"boards" in software development is widespread. But why is that? In a
lot of ways, a board is the go-to tool many of us have always used, and
asking "why" is like asking why we use a word processing application to
write a document. If we dig a little deeper, we might talk about
information radiators--that a board is fundamentally a communication
device, conveying status information to other folks in our organization.
Or we might discuss Boards as a prioritization tool, since it's easy to
rearrange cards as things change. And while these are certainly
important aspects, I believe that boards have a much more important use,
that is often forgotten: They provide a feedback mechanism for our work,
and help guide and verify our improvement efforts.

The use of cards on a board to visualize software development goes back
at least to the turn of the (21st!) century, and the work of folks like
[David J. Anderson](https://edu.kanban.university/users/david-anderson)
and [Mary and Tom Poppendieck](http://www.poppendieck.com/people.htm),
who were applying principles from lean manufacturing to software
development. Cards on a board were a good way of visualizing work, and
in particular, knowledge work that was difficult to go and look at.
Today the practice is ubiquitous, but I believe some of the original
context behind it has been lost.

The notion of a "card," or
[*kanban*](https://translate.google.com/?sxsrf=ALeKk01CTVci0TVZ-3Kh2ttjaaGC8xqrtQ:1586204201821&gs_lcp=CgZwc3ktYWIQAzIFCAAQgwEyAggAMgIIADICCAAyAggAMgIIADICCAAyAggAMgIIADICCAA6BAgAEEc6BQgAEJECOggIABCDARCRAkoQCBcSDDEwLTEwNWc4OWc2OUoMCBgSCDEwLTVnNWc4UMiFAVjklQFg05YBaABwAngAgAF5iAGTC5IBBDE0LjKYAQCgAQGqAQdnd3Mtd2l6&uact=5&um=1&ie=UTF-8&hl=en&client=tw-ob#view=home&op=translate&sl=ja&tl=en&text=kanban),
comes from Taichi Ohno's [Toyota Production
System](https://www.amazon.com/Toyota-Production-System-Beyond-Large-Scale/dp/0915299143/).
The original cards were used as a feedback mechanism to help regulate
inventory and improve flow, and they were the namesake of the system
that Toyota developed. There is a particular excerpt from Ohno's book,
which is often quoted:

> "It is said that improvement is eternal and infinite. It should be the
> duty of those working with kanban to keep improving it with creativity
> and resourcefulness without allowing it to become fixed at any stage."

For many teams, "the board" that everyone gathers around for 15 minutes
every day is fixed; cards might move, but the columns and the numbers
stay the same. If our visualization is static, then the process we're
using is probably static as well, and that means we aren't improving it!
This is a trap that is easy to fall into, but luckily it's easy to get
out of, as well--it only requires two steps: First, make sure that the
board is accurately modeling our current workflow, and then look for
places to improve that workflow. Iterating on these two things means
that our workflow will evolve, and our board may change along with it.

Modeling the Current Workflow
=============================

We want our board to be an accurate visualization of how work gets done.
In order to do that, we need to capture handoffs between people or
teams, places where we have to wait on approval, and promotion through
various environments (Dev, QA, Prod, etc). It's easy to visualize these
things by adding additional columns to the traditional "To Do, Doing,
Done" board layout, but it's important to recognize that we will
probably not get the model exactly right on the first try, and we'll
want to be able to make changes as we learn more about our process.

We also need to figure out where we start tracking our work, and where
we stop tracking it. Where you start depends on how your team is
operating; we like to start at the point that a card is ready to be
pulled and worked--that might be after a sprint planning meeting, but it
could be something much less formal--again, it's fine to make a guess.
Where to end is a bit more straightforward: The last column on the board
should almost always represent something in production, that is
available to real users. If we're handing work off to another team
before it gets to real users, we'll add another column to represent that
handoff! 

Both the start and end points of our board may change
over time, especially as the organization evolves, and teams become more cross-functional. We may want
to include user research or product-centric activities that come
"before" our start point, and research activities to get feedback once
our product is "live."  Similar to the way that the Theory of Constraints
extends up and down the entire supply chain, we our board might eventually
encompass an entire [value stream](https://en.wikipedia.org/wiki/Value_stream)
of our business.

A challenge can arise when the "information radiator" aspect of the
board is elevated above feedback, and multiple workflows are crammed
together in the same model. In these cases, it's usually possible to
meet both needs; individual teams need to have a clear visualization of
their own particular workflow for feedback, and folks in different parts
of the organization need a way to bring disparate information together
in a single, clear picture. By discussing those needs as separate value
propositions, we can often separate out their implementations, and make
sure that everyone is getting a tool that is fit for its purpose.

If you're struggling to get an initial model going, Dominica DeGrandis'
fantastic book, [Making Work
Visible](https://itrevolution.com/book/making-work-visible/) is full of
great content, and even has some team activities to help get things
rolling.

Improving the Workflow Based on Feedback
========================================

Once we have our initial model, we can use it for a little while, and
see how it feels. If we're using something like scrum, a sprint boundary
is a good trial period; if we aren't working on a cadence or we're doing
something like continuous retrospectives, we can take a look at our
process every time we move a card. Often it will be clear right away
that something isn't quite right, and we might need to adjust our model.
If there's an obvious gap (perhaps we forgot to model what happens when
work gets handed off to the QA team), we can just add a column. If
something feels off, but we're not sure what, our board can help guide
us.

In general, we want to look for places where work accumulates. Maybe
cards move relatively quickly through development, but "pile up" in the
QA column. If we see this behavior consistently, that's a great signal
to focus in on that part of the workflow, get the right people together,
and really understand what is happening--this is what [going to
gemba](https://www.lean.org/leanpost/Posting.cfm?LeanPostId=65) looks
like for knowledge workers. Sometimes we'll learn that our model is
off--It could be that the QA process goes quickly enough, but that the
work is waiting on a CAB approval before it can get deployed. In that
case we can adjust our model, and add an additional "CAB" column to our
board, after QA.

On the other hand, we may not see any obvious flaws in our model. If
not, then we've likely stumbled across a
[constraint](https://en.wikipedia.org/wiki/Theory_of_constraints) in the
system! Maybe QA is working as hard as they can, but they just can't
keep up. We can focus our improvement efforts around that constraint,
and try some experiments. Perhaps we can automate some of the testing,
or developers can pitch in and help with exploratory testing. Mike
Rother's [Improvement
Kata](http://www-personal.umich.edu/~mrother/The_Improvement_Kata.html)
is a great tool to help focus those efforts.

Iterating
=========

When we've found a bottleneck and started to experiment with our
workflow, we can use the board to help gauge whether our changes are
helping or not. We don't need anything fancy for this; we can count how
long cards are staying in the bottleneck column, and track that number
over time--counting manually and logging the results in a spreadsheet is
fine. If our improvement methods are helping, we'll see that number go
down. Eventually, the bottleneck itself might move--we may start off
seeing work build up in the QA column, and after a few weeks and a few
changes, we find that work is now accumulating in the Development stage.
Now we can change the focus of our improvement, and perhaps talk about
story slicing, or technical debt.

As our process changes, it's a good idea to check back and see if our
model is still accurate. If we *never* see cards in a certain column, or
find ourselves moving cards through multiple columns at the same time,
something might be amiss; it's often the case that our workflow has
evolved, but we haven't updated our model to match the new workflow.
Perhaps we've moved to a continuous integration approach, but we still
have a column for "Deploying." If we're skipping over that column for
every card, we might not need it anymore!

In general, we'll keep repeating the same basic steps in a [double loop
learning](https://en.wikipedia.org/wiki/Double-loop_learning)
pattern, something like this:

1.  Model the existing workflow

    a.  Are we missing a column?

    b.  Are there columns we no longer need?

2.  Improve the existing workflow, using our model for feedback

    c.  Where is work accumulating?

    d.  What can we try that might improve things at the bottleneck?

Tooling for Distributed Teams
=============================

I believe that, for most teams, the main purpose of a board is to
provide feedback for improvement efforts; the other benefits are
secondary. With that perspective, how should we choose a "board"
application for a distributed team?

First of all, we need a tool that allows us to make changes; we know
that our board layout will change, perhaps frequently. Modeling a
workflow is complicated, and we may try some things that don't work
out--we want to be able to experiment with our model without friction.

Second, the tool we choose needs to support board layouts that vary
between teams. Anything that imposes a common workflow is going to get
in the way of our modeling/improvement cycle, and different teams will
be in different places in that cycle.

Finally, the tool should make it easy for anyone on the team to move
cards between columns. The last thing we want is to be in a situation
where the work is done, but we're waiting to get the card moved on the
board--that will just introduce error when we're looking for
bottlenecks. It may sound silly, but I've seen all sorts of process get
built around moving cards, from huge automated workflows, to rules like
"only the project manager can touch the board." In these situations,
often the largest portion of the cycle time is just waiting for the card
to move after the work is done!

If we have these three things, the bells and whistles don't matter as
much; in particular, we don't need a rules engine, automatically
generated metrics, or complex change tracking. I've used
[Trello](https://trello.com/) for years, and have really liked
[Miro](https://miro.com/) more recently--both are go-to tools for me,
because of their simplicity and cost. I do think there is a risk of
complex, expensive tools dictating workflows rather than modeling them,
and I usually recommend that teams start with the simplest tool they
can, along with a spreadsheet for tracking metrics. That said, almost
any board can meet my three criteria with the proper configuration, and
a team focused on improvement can be successful even with mediocre
tools. I've used almost every board software out there
successfully\--even [Jira](https://www.atlassian.com/software/jira)!
