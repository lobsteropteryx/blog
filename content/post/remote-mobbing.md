+++
date = "2019-03-31T00:00:00+00:00"
draft = true 
title = "Remote Mob Programming"
+++

I've seen a lot of skepticism about mob programming lately, so I wanted to share an experience report and some data about my team's use of it.  The TLDR is that we saw significant improvements in cycle time and defect rate.

## About Us

We were a fully-distributed team of seven, working across four timezones in the U.S.  We operated pretty close to standard XP, planning out enough work for a week at a time, with daily standups and a weekly retrospective.  We pair programmed on all our work, and each card additionally went through a typical pull-request/code review phase, to be reviewed by another pair on the team.  All of our code was test-driven, and our build and deploy processes were all automated.  We did not estimate our cards beforehand, but focused on slicing stories into 1-3 day chunks of value, and using historical cycle times to predict delivery.