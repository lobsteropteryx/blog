+++
date = "2016-11-24T00:00:00+00:00"
draft = false
title = "Vim"
+++

So I'm at [SCNA](http://scna.softwarecraftsmanship.org/), participating in the [Code Retreat](http://coderetreat.org/).  If you've never done one, the idea is that you pair up with another developer, and work on [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway's_Game_of_Life) for 45 minutes, using TDD.  At the end of each 45 minute block there's a short retrospective, then you pair with a different person and start all over again.  Subsequent blocks might also have restrictions, such as "no passing primitives across function boundaries."

It's the second or third round of the day, and I'm pairing with a friend of mine.  We decide to work in Python on his macbook, using Vim (his editor of choice).  I am neither a Python nor a Vim expert, but I'm comfortable enough with both that I figure I won't be too much of a drag on our productivity.  Somewhere in there they announce the restrictions for the round, which are:

* We should ping-pong, with one person writing a test, and the other writing some production code, then swapping
* We are't allowed to speak or type messages to one another (!)

Now my pairing partner is one of my best friends, so that makes things easier.  But he's also a top-notch developer, so I'm feeling a little bit nervous about working in an unfamiliar environment--just that professional pride thing, where you don't want to look incompentent in front of someone you respect.  Still, I reassure myself--I've done a few small projects in Python, and I have a decent command over basic Vim motions; I should be okay.

We spend the last couple minutes discussing our approach, and dive in.

He starts off driving, writing a simple test, and passing me the laptop.  I enter edit mode, start typing, and...well, maybe my Vim chops aren't as good as I thought--something's wrong.  My partner gesticulates and does a quick couple of keystrokes; I go back to editing, and things seem to be working.  I finish a bit of production code, write the next test, and pass it back.

He makes the test pass, thinks for a bit, and writes another test.  My turn again.

This time around I'm extra deliberate--it's just `ESC` and `I`, right?  I start typing, but something's wrong--the cursor is flying all over, and--can this be true?  Yes, I just deleted a bunch of code.  The same frantic gestures from my buddy, and then what I guess to be the said gestures translated into Vim, and things are working again.

This goes on for a while, and eventually we get into a routine:  He writes his code and tests, and then makes sure to put Vim into what I assume to be "moron mode" before allowing me to touch the keyboard.  I'm not feeling too good about myself, but we manage to make progress nonetheless; the 45 minutes fly by, and they announce the end of the round.

The room explodes with conversation after almost an hour of silence, and my partner and I chat about the problem, our approach, and what our next steps would've been. At some point I feel obligated to apologize--"I'm REALLY sorry, it's clear that my Vim skills aren't nearly as good as I thought.  But please enlighted me, what keystrokes were you entering every time you passed the computer to me?"

"Oh," he says, "I just kept forgetting to take the keyboard out of Dvorak mode."

