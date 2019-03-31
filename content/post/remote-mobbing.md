+++
date = "2019-03-31T00:00:00+00:00"
draft = true 
title = "Remote Mob Programming"
+++

I've seen a lot of skepticism about mob programming lately, so I wanted to share an experience report and some data about my team's use of it.  The TLDR is that we saw significant improvements in cycle time and defect rate.

## About Us

We were a fully-distributed team of seven, working across four timezones in the U.S.  We operated pretty close to standard XP, planning out enough work for a week at a time, with daily standups and a weekly retrospective.  We pair programmed on all our work, and each card additionally went through a typical pull-request/code review phase, to be reviewed by another pair on the team.  All of our code was test-driven, and our build and deploy processes were all automated.  We did not estimate our cards beforehand, but focused on slicing stories into 1-3 day chunks of value, and using historical cycle times to predict delivery.  Each card was demo'd to our product owner/end user before being deployed to production.

In general, we would have 3-4 cards in progress at any given time; all work was code reviewed, and we were shipping new code to production at least a few times a week.

## After Mobbing

When we started mobbing, we would only have a single card in progress at a time; everyone would work the same card together, starting with 2-3 people in the morning, moving up to all seven for most of the day, and tapering back down to 2-3 in the evening.  We set aside time at the end of our overlapping hours for a daily retro.  Our previous "ceremonies" of standup, retro, and planning sessions all went away, but we would usually pause whenever new members joined the mob, to give a quick recap and bring everyone up to speed.