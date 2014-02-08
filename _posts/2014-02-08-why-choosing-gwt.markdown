---
layout: post
title:  "Why choosing GWT ?"
date:   2014-02-08 19:00:00
Tags: [Google Web Toolkit, GWT]
Categories: [GWT]
---


In the following article I will give you some of the reasons that pushed me to use GWT in my company. I will go from the most important to the less important (so that you can only read the first one).

## No javascript developer

In my opinion, if in your company, you do not have JavaScript developers but you need to do a professional web application. Go for GWT.

Java and Swing devs will learn it quicly (even better if they have experience in MVC). Plus GWT packages a lot of good practices to optmize your web site performance.

## Statically typed

It is the strength (and the weakness) of GWT. By coding in java you benefit from all the good stuff coming with it.

- very powerfull IDEs like [Eclipse](https://www.eclipse.org/) or [IntelliJ](http://www.jetbrains.com/idea/) with refactoring capacity, static code analysers [checkstyle](http://checkstyle.sourceforge.net/), [findbugs](http://findbugs.sourceforge.net/) etc.
- natural OOP (even if js is becoming more and more OO)
- unit tests fwk ([Junit](http://junit.org/)) + coverage ([Emma](http://emma.sourceforge.net/)) + mocking ([Mockito](https://code.google.com/p/mockito/))

Those advantages become critical when your application becomes big ( +100k lines). Refactoring a big application in JavaScript is a nightmare in my experience and it gets even worse when more than 5 people are working on it ! If anyone has experience on that topic please share it in the comments.

## Use JavaScript !

That's the beauty of it. If you need a js lib or component, simply use it ! Have a look to my article on the latest news about [JSNI](/2014/02/02/GWT-futur-javascript-interop.html). Use GWT as much as you want, simply for a widget or for your whole application, it is up to you, you can mix the two.

# Multi-browser / mobile support

Even if it is also true for some js fwk. Browser support is quite transparent in GWT. And you can count on GWT guys to include the best solution for each browser.

And with a good MVC implementation mobile comes easily. Have a look to [m-gwt](http://www.m-gwt.com/) by [Daniel Kurka](http://www.daniel-kurka.de/)

## Open source / free

No need to pay any license / support ! For small companies I beleive it can be very important.

I am sure some of you have other good reasons. Please share them in the comments and I will update the article.