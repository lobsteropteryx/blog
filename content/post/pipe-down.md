+++
date = "2017-05-03T00:00:00+00:00"
draft = false
title = "Pipe Down"
+++

As a full-time remote developer, it goes without saying that an internet connection is a requirement; every developer will be pushing and pulling from source control, chatting via slack, and performing ~~hundreds~~ thousands of package installs and/or stack overflow searches.

I also spend a lot of time using some sort of screensharing tool--all of our production code is written in pairs over screenhero, and we do our daily standups, architecture discussions, and watercooler sessions over Google Hangouts.

I've been curious about how much bandwidth is really necessary to work remotely as a developer; while working from home has a ton of obvious advantages, needing to work in the same place you live can be a big limitation, especially when it comes to connectivity.

In most parts of the United States, you don't have to go very far out into the county before your internet options become very limited.  Cable is almost out of the question even a few miles outside of town; DSL can provide decent bandwidth, but it falls off very quickly--while you might get 20 Mb/s within a mile or two of a repeater, by the time you get 8-10 miles away, you're down to 1.5 Mb/s.  Line-of-sight wireless and satellite can be unreliable, and often have strict data limits on total data upload.

Many cellular providers offer unlimited data plans, with bandwidth throttled past a certain point--typically the speed drops to somewhere around 0.15 Mb/s after the first 10 or so GB.  As an experiment, I decided to try working with my laptop tethered to my phone; this allowed me to get a good measurement of the bandwidth and total data usage, as well as a qualitative idea of the effect of throttling past the cutoff point.

## Tethering with Android
Tethering an Android phone is super easy--Just go to `settings` -> `Wireless & networks` -> `More...` -> `Tethering and portable hotspot`.  You can set up whatever SSID and authentication settings you like, and just check the checkbox to enable wireless access.  Then connect your laptop via your normal wireless UI.

## Data Usage
Based on ~6 hours of sharing via screenhero, and another hour of multi-user Hangouts, I used somewhere around 8GB of data:

<img src="/images/data-usage-1.png" alt="data-usage" style="width: 400px;"/>

Screensharing performance was great at full speed (~ 8Mb/s down, 1Mb/s up), but once I hit the throttle point, the bottom fell out:

<img src="/images/throttle.png" alt="throttle" style="width: 400px;"/>

Both screenhero and hangouts will use adaptive quality, to some extent; I didn't try rolling the quality settings down, but I'll do this experiment again, and see how much difference those settings make.
