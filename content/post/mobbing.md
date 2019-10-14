+++
date = "2019-10-10T00:00:00+00:00"
draft = true 
title = "Mob Programming"
+++

## Our Path To Mobbing

This is an experience report from our team, who first dabbled with mob
programming two years ago; since then we have adopted it as part of our core
culture, and continue to spread it throughout our organization and clients.  We
talk a bit about who inspired us, how we convinced our leaders to let us try
it, and where we ended up.  We also show some of the data we collected along
the way, and talk about the advantages we've seen, the challenges we've
encountered, and some of the ways we've adapted.

### Starting Out 

We learned about the idea of [mob
programming](https://en.wikipedia.org/wiki/Mob_programming) from hearing [Woody
Zuill](https://woodyzuill.com/) speak at a conference.  Most of our team was
there at that talk, and we were all excited by the idea; after some
conversations in person, and later on twitter, we were convinced that we needed
to give it a try.

As a team practicing [XP](https://en.wikipedia.org/wiki/Extreme_programming),
we had a lot of positive habits; we were already comfortable pairing, and had
a deep culture of quality, with strong support from the rest of the business.
At the same time, we could see some challenges; we weren't delivering as fast
as we thought we should be, and despite both pairing _and_ code reviews, our
defect rate was quite high.  Even though we were pair-swapping frequently, we
were still seeing a lot of knowledge silos, and the overall quality of the code
base didn't seem to be improving. We thought that mobbing might help shore up
some of those gaps, and we came back from the conference ready to pitch the
idea to our manager, "Dave."

### Convincing Management

Dave had helped drive a lot of the team culture, and had been working with XP
for a long time; he had also been with us at Woody's talk, and seemed as
excited as we were about the idea.  So we were surprised, and a little
disappointed, at how much resistance we got when we pitched the idea of the
team trying out mobbing for an iteration.

It's easy to work through things when there is mutual respect, so we went back
and forth on the topic, and tried to understand his concerns.  We were a team
of seven at the time, and the main issue boiled down to an efficiency argument;
when push came to shove, it was just hard to believe that seven people working
together wouldn't be slower than 3-4 pairs working in parallel.

At the time, we were working on a one-week cadence, and that was more than Dave
felt comfortable committing to.  He was willing to let us try out the idea on
a single _story_, so we started there.  Knowing that he was both reasonable
_and_ uncomfortable, we figured that we would need rely on data to make our
case.

We were already tracking our throughput (from the time a pair started the work
until it was usable in production), as well as our defect rate, so we pulled
a few months of historical data on those two metrics to server as a baseline.
Then the team talked through some of Woody's suggestions and techniques, and we
pulled our first card as a mob just before Haloween!

### Seeing Results

Although we couldn't really make any statistical arguments, the first card
moved quickly enough for us to ask for a second.  The second one moved as well,
which led to a third, and soon we had completed our first mobbing iteration!
That was a watershed moment; the team was still very excited, and the numbers
looked good, so it was easy to get everyone's buy-in to mob for another
iteration--including our manager.

We worked as a seven-person mob for the next three months; we continued to
track our two initial metrics and make them visible, and the results were good
enough that we weren't asked to justify the approach again. As the weeks went
on, a clear pattern emerged:  Our defect rate went down considerably, but our
throughput hardly changed at all; we were delivering at the same rate, but with
*significantly* fewer bugs.

## The Data

Although we probably don't have the data necessary for a rigorous comparison,
even the rough, monthly numbers tell an interesting story:

<img src="/images/mob-data.svg" width=100%/>

There are some details that don't show up in the chart above, that are
important when interpreting the data--the nature of the work, the source of
defects, and the way that throughput was calculated.

### The Work

The team had been working together for over a year building custom Extract,
Transform and Load (ETL) applications, each consuming data from a different,
third-party data source.  A typical application would retrieve the data, apply
some complicated business rules and transformations, and store the results for
future analysis.

Although we had done similar work before, we had yet to find patterns or
abstractions that felt generally applicable across data sources; there was
a level of complexity in the code base that seemed unnecessary, and enough
differences in the data sources and business rules that each new source felt
like a new problem, even though the overall goal was familiar.

When we began to mob, we were working on another, similar application, with
a new data source.  The data was from an entirely different domain, with new
business rules that we weren't familiar with, and we were looking for a better
way of designing these processes.

### Defects

The spike in defects between August and October corresponds to the development
and release of a previous application, consuming an entirely different data
source; it is roughly comparable to the November through January timeframe that
we spent mobbing.

It's important to point out that the bugs that were found in November and
January were problems with previous applications, that had been built without
mobbing.  And although we don't show the data here, the application that was
built using mob programming has been in production for almost two years without
a single defect!

### Throughput

The throughput numbers in the chart above include the work done to fix
defects.So while the the throughput appears to be about the same whether we
found eight defects or zero, there is a tremendous difference in the amount of
value delivered; in months with a high defect count, most of the "throughput"
is actually rework, fixing bugs found in previously-delivered features.

## What We Learned


