+++
date = "2013-06-09T00:00:00+00:00"
draft = false
title = "So Many Ways..."
+++


<blockquote>
"There are too many languages today, and it would be a good idea to choose just one."
<footer>Richard Feynman, Nishina Memorial Lecture "The Computing Machines in the Future," Tokyo, Japan, 1985 (from <a href="http://www.amazon.com/Selected-Papers-Richard-Feynman-Commentary/dp/9810241305/ref=sr_1_1?ie=UTF8&qid=1370793875&sr=8-1&keywords=selected+papers+of+richard+feynman">Selected Papers of Richard Feynman</a>)</footer>
</blockquote>

A couple weeks ago, I started thinking about writing a simple blogging framework, and, I guess, a blog?  Of course, there is approximately zero need for more blogs, or more software for <i>making</i> blogs--there are plenty of great platforms out there, if all you want to do is write down some thoughts that no one will ever read.  But there's something about working on a project where you're both the developer <i>and</i> a user--it's good for you.  I moved on to thinking about what technologies to use.

It's kind of a weird question:  Should I reinvent the wheel in Ruby?  Python?  PHP?  ASP.NET?  It's something that people actually discuss!


(I'm not even going to bother putting a link there).


For the past several years, I've worked on a daily basis with:

-C#, ASP.NET and Dojo for web development;
-Subversion for source control;
-Stored procs in an Oracle database for persistance;
-IIS for my webserver.


You can always go deeper, and get better at the things you know--I've done zero work using MVC 4 or Entity Framework, and I've only scratched the surface of what Dojo has to offer.

I remember a Physics professor of mine, who used to work for IBM.  We were working on an electronics project, and something wasn't behaving correctly.  We went to his office, and he whipped up some circuit simulations...in Fortran.  His advice was to learn one language, and to learn it really well, so that you have a useful tool at your disposal whenever you need to, you know, actually <i>do</i> something.

On the other hand, lots of <a href="http://pragprog.com/book/tpp/the-pragmatic-programmer">smart people</a> advocate learning new languages frequently, as a way to both keep up with new technologies, and expand your way of thinking about programming in general.  And when the <a href="https://en.wiktionary.org/wiki/if_all_you_have_is_a_hammer,_everything_looks_like_a_nail">only tool you have is a hammer...</a>

In the end, I decided to dive in head first, and use a totally different stack, that I have very little familiarity with.  This site is built using Python, Django and jQuery, on top of a Postgresql database served from Apache, and the code is stored in a Git repository.

If you're reading this, it works.
