+++
date = "2013-10-10T00:00:00+00:00"
draft = false
title = "Homework"
+++

The other day I was called upon to help a 5th grader with her homework.  She had gotten the first two problems right, but was having some difficulty with the third one:

![](/images/homework.gif)

I can't help but wonder who chose that problem, and what their intention was; if it's supposed to be a way to get kids thinking about algorithms and the idea of proof, that's great.  Probably most kids would start off okay--you find all the paths and pick the shortest one.  But how do you know there's not a shorter one, that you didn't find?  Can you even expect to find all the possible paths?  I think it's probably intuitive that the number of paths might increase faster than the number of nodes, but how much faster?  And what about cycles?

And maybe there are geometric solutions--if you drew a line between the start and end points, are there any generalizations you can make about the paths (i.e., paths that intersect the line versus those that don't)?  Could you work it out with a compass?  But the segments on the paper weren't even drawn to scale, so compass-work was out of the question.  In the end, maybe someone just thought that it was a nice way to make a kid add up a bunch of numbers--"oh, it's like, four addition problems in one!"

That's all probably a bit much for a 5th grader, and this one was certainly not interested in my flights of fancy--what she wanted to know was, did she get the right answer?  When I told her that I didn't know, she thought I was making fun of her, but when I told her that I was pretty sure that her teacher didn't know the answer either, she started to wonder.  I mentioned that, in my opinion, it was a very hard problem for a fifth grader, but that while I didn't <i>know</i> the answer, I knew how to <i>find</i> it.  (That spun off into another conversation entirely, about the nature of education; I might write something on that topic another day).

I thought that this would make a pretty good programming <a href='http://codekata.pragprog.com/'>kata</a>, so I went to work.  I was at least passingly familiar with <a href='http://en.wikipedia.org/wiki/Dijkstra%27s_algorithm'>Dijkstra's algorithm</a>, but I decided to see if I could iterate my way to a solution without looking it up.  I started out trying to represent the nodes and segments, and then figured that I wanted something recursive, to keep comparing new segments at each new node.  I went on this way for a while, but the results weren't very promising, partially because I'm a TDD novice, and partially because I'm just bad; in the end I gave up, and looked up the algorithm (See--bad).

But I'm not sure that iteration is always the best solution to this type of problem; I think it has a place, particularly if you're implementing a known algorithm (my <a href='https://github.com/lobsteropteryx/GardenClub'>implementation</a> is probably full of bugs), but for solving a new problem, I'd probably want to go to pencil and paper first.  In any case, that topic has already been <a href='http://ravimohan.blogspot.com/2007/04/learning-from-sudoku-solvers.html'>discussed</a> plenty, by smarter people than me.  I do wonder what <a href='http://www.cs.utexas.edu/users/EWD/transcriptions/EWD04xx/EWD498.html'>Dijkstra himself</a> would have to say about it, though.

