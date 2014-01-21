---
layout: post
title:  "Javascript object comparison"
date:   2012-12-12 18:19:00
Tags: [javascript]
Categories: [javascript]
---


Hello everyone,

Today, working on an elegant message box system for our Sencha Touch mobile application I asked myself the following question : How does the === operator works when comparing objects ?

Well I found this article useful : [http://javascript.about.com/od/byexample/a/objectoriented-compareobject-example.htm](http://javascript.about.com/od/byexample/a/objectoriented-compareobject-example.htm)

The fundamental point is, when you assign an object to a var in javascript, this var is on only a reference to the object. The  === comparator will compare the object your reference is pointing on. The following code will only display 'true' values.

{% highlight javascript %}
var ob1, ob2, ob3;
ob1 = {prop1: 'a', prop2: 'b'};
ob2 = {prop1: 'a', prop2: 'b'};
ob3 = ob1;
if (ob1 === ob2) alert('false');
if (ob1 === ob3) alert('true');
ob3.prop3 = 'c';
if (ob1 === ob3) alert('true');
if ('c' === ob1.prop3) alert('true');
{% endhighlight %}

Now if you want to re-write an equal method in javascript for 2 different objects having the same properties you should have a look at the following thread on Stackoverflow :

[http://stackoverflow.com/questions/1068834/object-comparison-in-javascript](http://stackoverflow.com/questions/1068834/object-comparison-in-javascript)