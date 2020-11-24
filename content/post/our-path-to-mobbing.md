---
title: "Our Path to Mobbing"
author: "Ian Firkin"
date: 2019-10-21
---

Note: This post was originally published on our company site, which has since been taken offline.

This is an experience report from our team, who first dabbled with [mob
programming](https://en.wikipedia.org/wiki/Mob_programming) two years ago;
since then we have adopted it as part of our core culture, and continue to
spread it throughout our organization and clients. We talk a bit about who
inspired us, how we convinced our leaders to let us try it, and where we ended
up. We also show some of the data we collected along the way, and talk about
the advantages we’ve seen, the challenges we’ve encountered, and some of the
ways we’ve adapted.

Many of the ideas here are also covered in the [Mob
Programming](https://leanpub.com/mobprogramming) book; it was essential in
guiding us through our initial adoption of mobbing, and if this article is
interesting to you, the book explores all of these topics in more detail.

## Starting Out

Like many folks, we learned about the idea of mobbing from [Woody
Zuill](https://woodyzuill.com/).  Most of our team heard him speak at
a conference we were attending, and we were all excited by the idea.  After
talking with him in person, and later on Twitter, we were convinced that we
needed to give it a try.

As a fully distributed team, we would be mobbing over a shared video call with
screen sharing, and our communication would be more constrained when compared
to working face-to-face. Woody was quite emphatic in his talk and book that mob
programming could apply to distributed teams just as well as colocated ones,
and we felt that we would be able to navigate the communication issues.

We had already been working together for a couple of years, practicing
[XP](https://en.wikipedia.org/wiki/Extreme_programming), and we had built a lot
of positive habits; we were very comfortable with pair programming, and had
a deep culture of quality, with strong support from the rest of the business.
At the same time, we could see some challenges; we weren’t delivering as fast
as we thought we should be, and despite both pairing and code reviews, our
defect rate was quite high. Even though we were pair-swapping frequently, we
were still seeing a lot of knowledge silos, and the overall quality of the code
base didn’t seem to be improving. We thought that mobbing might help shore up
some of those gaps, and we came back from the conference ready to pitch the
idea to our manager.

### Convincing Management

Our manager had helped drive a lot of the team culture, and had been working
with XP for a long time; he had also been with us at Woody’s talk, and seemed
as excited as we were about the idea. So we were surprised, and a little
disappointed, at how much resistance we got when we pitched the idea of the
team trying out mobbing for an iteration.

It’s easy to work through things when there is mutual respect, so we went back
and forth on the topic, and tried to understand his concerns. We were a team of
seven at the time, and the main issue boiled down to an efficiency argument;
when push came to shove, it was just hard to believe that seven people working
together wouldn’t be slower than 3-4 pairs working in parallel.  At the time,
we were working on a one-week cadence, and that was more than our manager felt
comfortable committing to. He was willing to let us try out the idea on
a single story, so we started there. Knowing that he was both reasonable and
uncomfortable, we figured that we would need to rely on data to make our case.

We were already tracking our throughput (from the time a pair started the work
until it was usable in production), as well as our defect rate, so we pulled
a few months of historical data on those two metrics to serve as a baseline.
Then the team talked through some of Woody’s suggestions and techniques, and we
pulled our first card as a mob just before Halloween!

### Seeing Results

Although we couldn’t really make any statistical arguments, the first card
moved quickly enough for us to ask for a second. The second one moved as well,
which led to a third, and soon we had completed our first mobbing iteration!
That was a watershed moment; the team was still very excited, and the numbers
looked good, so it was easy to get everyone’s buy-in to mob for another
iteration–including our manager.

We worked as a seven-person mob for the next three months; we continued to
track our two initial metrics and make them visible, and the results were good
enough that we weren’t asked to justify the approach again. As the weeks went
on, a clear pattern emerged: Our defect rate went down considerably, but our
throughput hardly changed at all; we were delivering at the same rate, but with
significantly fewer bugs.

## The Data

Although we probably don’t have the data necessary for a rigorous comparison,
even the rough, monthly numbers tell an interesting story:

<img src="/images/mob-data.svg" width=100%/>

There are some details that don’t show up in the chart above that are important
when interpreting the data–the nature of the work, the source of defects, and
the way that throughput was calculated.

### The Work

The team had been working together for over a year building custom Extract,
Transform and Load (ETL) applications, each consuming data from a different,
third-party data source. A typical application would retrieve the data, apply
some complicated business rules and transformations, and store the results for
future analysis.

Although we had done similar work before, we had yet to find patterns or
abstractions that felt generally applicable across data sources; there was
a level of complexity in the code base that seemed unnecessary, and enough
differences in the data sources and business rules that each new source felt
like a new problem, even though the overall goal was familiar.

We started our mobbing experiment by kicking off another, similar application,
with a new data source. The data was from an entirely different domain, with
new business rules that we weren’t familiar with, and we were looking for
a better way of developing these processes.

### Defects

The spike in defects between August and October corresponds to the development
and release of a previous application, consuming an entirely different data
source; it is roughly comparable to the November through January timeframe that
we spent mobbing.

It’s important to point out that the bugs that were found in November and
January were problems with previous applications, that had been built without
mobbing. And although we don’t show the data here, the application that was
built using mob programming has been in production for almost two years without
a single defect!

### Throughput

The throughput numbers in the chart above include the work done to fix defects.
So while the throughput appears to be about the same whether we found eight
defects or zero, there is a tremendous difference in the amount of value
delivered; in months with a high defect count, most of the “throughput” is
actually rework, fixing bugs found in previously-delivered features.

## What We Discovered

During our first three months working together as a single mob, we saw some
incredible improvements in the way our team worked together, the speed of our
delivery, and the quality of the systems that we built. Looking back, we’ve
identified two things that stand out as drivers for those improvements: Better
transfer of learning and culture across the team, and reduced wait time for
things like code reviews and merges.

### Learning and Culture

As mentioned before, we started out working in pairs, and we would “swap” pairs
after each completed story. The idea was to spread knowledge across the team,
and foster collective ownership of the code base. In practice, we were still
seeing silos of knowledge, and different pairs might tackle similar problems in
very different ways. In addition, certain folks worked especially well
together, and it was easy for teammates to get “left behind;” sometimes the
people who needed the most support were the last to get it.

Once we started working together as a single mob, these issues largely went
away. If anyone in the group was struggling, we could all pitch in and help; if
there was a behavior problem, it was much easier to recognize and talk about
with the whole team involved. We were also able to spread new ideas and
practices across the team much more quickly; it’s probably not an exaggeration
to say that we leveled up more in three months of mobbing than we had in the
previous year of pairing. In the end, while pair swapping was certainly helpful
in some ways, mobbing proved to be significantly more effective at achieving
the same goals.

### Code Reviews and Trunk-based development

Before we started mobbing, we were operating as 3-4 distinct pairs; each pair
would work their own story on a branch. Once a pair felt the story was
complete, they would issue a pull request, and wait for another pair to review
their code. Typically there would be some back-and-forth discussion between the
pairs, and finally their code would be merged into the trunk. While we tried to
slice our stories as small as possible, there were usually a few days between
merges, and we were dealing with common problems: Large pull requests,
difficult merge conflicts, and multiple cycles of rework.

When we started to mob, we cut out the code reviews; right from the start it
felt very freeing. Discussions about technical decisions and approach were
happening much earlier, and asking questions of our users or business owners
was less of a bottleneck since they weren’t being pulled in multiple
directions. Best of all, there was no waiting–once we agreed that the users’
needs were met, we could push to production and pull the next card!

We did have some doubts about this at first; code reviews have been studied
quite a bit, and are widely considered a best practice. But right away the
quality of the code produced by the mob “felt” better to the team, and this was
born out over time by our defect rate–it fell to zero.

Since we were working together, there was no longer any need for branching; we
could merge into trunk as often as we wanted, without fear of conflict. This
also helped us slice our work smaller; more than once, someone on the team
would ask “Can we release this to production now?” Often the answer would be
yes, and we would be able to split one story into two smaller ones, and deliver
useful functionality to users sooner.

## Challenges

Although mobbing was undoubtedly a success story for us, we did make some
missteps, and encountered some challenges along the way. The biggest issues we
had to tackle were making sure everyone had a voice and felt safe speaking up,
and learning how to work in a sustainable way and not burn people out.

### Giving Everyone a Voice

When we started mobbing, everyone was working together on the same thing, at
the same time; the team had different skill sets and experience, not to mention
personalities! One trap that we fell into a few times was having a few voices
dominate the conversation. This can happen when pairing or in any meeting, but
with seven people on a call for the whole day, it could be especially
problematic.

We had a couple of advantages as a team that helped us; we were in a safe,
optimistic place going in, and folks were mostly comfortable speaking up when
something felt wrong. In addition, we had some talented facilitators on the
team, who helped navigate conversations when things got heated, and made sure
that we were considering everyone’s perspectives.

We still caught ourselves in some bad habits, and things occasionally got
tense; the team as a whole was really great at calling out those situations
when they occurred, and helping us all be accountable. When things started to
go awry, someone would always speak up and remind us to take a break, be
disciplined about using strong style pairing, or just to be respectful.

### Working Sustainably

One of the most interesting things about working in a mob was seeing how
different people on the team reacted to it. For some folks it was energizing
and exciting, and for others, it was exhausting! A whole new set of norms were
needed to accommodate everyone’s needs, and to help keep the whole experience
positive and sustainable; the biggest factors were ensuring that people felt
comfortable taking breaks when they needed to, and determining what work could
be done individually versus by mobbing.

Establishing some guidelines worked well for us; for example, we knew that we
wanted coding activities to happen in a mob setting, and we really wanted
everyone to be involved in conversations with customers. On the other hand, we
found that trying to do research or learning about new technologies as a mob
was uncomfortable and ineffective; instead, we would often “split up” for an
hour or two, then come back together and talk through what we had learned as
a team. And while we did have a daily “team learning” time with the entire
group, we also made sure that personal study time was available to everyone as
well.

In the end, our strategy to keep everyone happy and engaged was to foster
trust, let people do what they needed to, and to assume that everyone was
working to move the work and the organization forward.

## Today

Our initial experiments with mobbing that we describe here happened almost two
years ago; since then, we’ve continued to practice mobbing and evolve our
process.  The members of our original mob have moved and evolved as well, and
eventually made their way into four different, cross-functional teams.  We have
brought mobbing with us into these teams, and continue to see the benefits,
along with new challenges. Overall it has been an amazing experience for us,
and has helped reinforce collaboration as one of our core values as an
organization.


