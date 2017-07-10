+++
date = "2013-09-14T00:00:00+00:00"
draft = false
title = "Unconventional"
+++

I’ve been thinking a lot lately about the idea of <a href='http://en.wikipedia.org/wiki/Convention_over_configuration'>convention</a> in software development.  The more I’ve worked with the ASP.NET MVC framework and nHibernate, and dabbled with Django, the more I’ve been able to see the benefits of designing and coding this way.  It’s also made me consider my own biases and background, and realize why I’ve struggled with the idea in the past.

I started out as an embedded developer, and for a long time, everything I wrote was in C or some flavor of assembler.  That’s a very explicit way of coding, not just malloc’ing and freeing, but often managing the physical address space itself.  That’s not to say that there aren’t conventions in lower-level languages, but they are more like <a href='http://www.crockford.com/javascript/private.html'>that</a> and Captial-C Constructors are in javascript:  mostly-agreed-upon ways to make implicit things clearer, and avoid common pitfalls that are instrinsic to the language.*

So I feel like there’s a difference; if I give my C (or javascript) functions unique and delightful names, my code will still build and run, it will just be hard to understand.  But if I don’t set up my routing correctly in an MVC application, or specify my mapping correctly when using an ORM, my application just won’t work, and I’ll be left trying to figure out what went wrong.   When things are happening automagically, it can be a real killer trying to track down which stupid thing you did (or didn’t do); I had been somewhat frustrated working with those tools before, and I finally realized what my problem was:  I was coding along, just automatically assuming that things like variable (or function, or table) names <i>didn’t matter.</i>

And of course, when you’re working with tools that are convention-based, those things matter a great deal.  In fact, I think that’s really the point:  Using conventions means that you’re assigning meaning to things implicitly, in ways that aren’t enforced by the language (or framework) itself.   In return, you get a whole bunch of stuff done for you, for free.  And of course, the farther you go, the closer those conventions come to being almost a <a href='https://github.com/jagregory/fluent-nhibernate/wiki/Conventions'>higher-level language</a> themselves.

*  Maybe that’s why developing in javascript, of all things, feels <a href='http://www.hanselman.com/blog/JavaScriptIsAssemblyLanguageForTheWebSematicMarkupIsDeadCleanVsMachinecodedHTML.aspx'>sort of familiar</a> to me.  And by “familiar,” of course I mean “bad.”

Just <a href='http://www.amazon.com/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742'>kidding</a>.



