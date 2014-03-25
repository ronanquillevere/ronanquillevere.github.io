---
layout: post
title:  "Using Guice with Jersey (2.6)"
date:   2014-03-25 23:59:00
Tags: [Guice, Jersey, APO, Dependency Injection, IOC, ]
Categories: [Jersey]
---

Hi everyone,

In my company we love 2 things (among others), JAX-RS ([Jersey](https://jersey.java.net/)) and dependency injection ([Guice](https://code.google.com/p/google-guice/)). We are used to those 2 implementations and we want to keep using them. But unfortunatly they do not really like each other ... But things are getting better. The guys from Jersey have implemented a great feature (even if still far from being perfect) which is a kind of injection bridge.

Basically it lets you use your own dependency injection framework within Jersey. You are not forced to use [HK2](https://hk2.java.net/2.3.0-b03/). In our case we want to use Guice.

But the setup of this bridge can be tricky, so one of my collegue from MYCOM decided to share the way we are able to make it work. 

Please have a look to our [repository](https://github.com/mycom-int/jersey-guice-aop) where we share the code to make Jersey 2.6 and Guice work together. Do not hesitate to comment the code here.