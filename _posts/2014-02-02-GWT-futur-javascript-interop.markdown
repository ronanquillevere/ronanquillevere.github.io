---
layout: post
title:  "Next generation javascript Interoperability with GWT 3.0"
date:   2014-02-02 19:00:00
Tags: [Google Web Toolkit, GWT, javascript, interoperability, JSNI, js, wrapper]
Categories: [GWT]
---

Hello everyone,

Today I am going to talk about javascript interoperability with GWT and more precisely about the new features that will be introduced in futur version of GWT. I believe this is a major topic, one that could lead to an even greater success or that could lead to the end of GWT. I have recently read the spec for GWT 3.0 and I want to share some of its content with you. You can find all the source links at the end of this article.

# A little (personal) history

When I started my new job 1 year ago, as a tech lead, I was responsible for choosing the best technology in order to develop our new product. I am not going to detail the reasons that made me choose GWT here (it could be part of another article) but what I can say is that I really hesitated and one of my concerns was the actual (GWT 2.5) javascript interoperability.

I really doubted our capacity to integrate javascript components/libraries easily. The javascript community is an incredible community and I discover incredible frameworks, libraries, widgets everyday. I really wanted to be able to use those great components in our product. I had the feeling that the trend was to go 100% javascript. After using [Sencha Ext Js](https://www.sencha.com/products/extjs/) and [Sench Touch](http://www.sencha.com/products/touch/) in my previous company and witnessing the incredible rise of [Angular.js](http://angularjs.org/) I was getting scared to choose GWT. So if we were going to go for GWT, at least, we needed to be able to integrate javascript lib easily !

During my previous experiences, I used JSNI a lot with GWT, in both directions, if I can say that, from GWT to javascript (which is the most common use case) and from javascript to GWT.

####GWT -> Javascript
When I was working for [one2team](http://www.one2team.com), we integrated the [Highcharts](www.highcharts.com) library inside our GWT application and we built our own wrapper (some of this work can be found [here](https://github.com/one2team/highcharts-serverside-export)). It was before moxie commited their own [wrapper](http://www.moxiegroup.com/moxieapps/gwt-highcharts/). We also coded the first backend rendering engine for highcharts.  Now, for server export, you can probably do better by using Phantom.js.

####Javascript -> GWT
I also wrapped a [GXT](http://www.sencha.com/products/gxt/) customized live combo into a javascript component that was populated from jsp pages. 

<!-- In today's industry, I feel like we are more and more challenged to deliver products with a high level of quality, really fast, so re-coding everything is not an option. Plus, I strongly believe that open-source code generally leads to better quality than what you can do internally. It is like doing a code review with many people ! 

One of the strong point of GWT is that, by using Java, you can really properly unit test / cover your code. When you are building B2B applications, your clients expects a high level of quality (good [MTBF]() for example). Even if javascript tools for testing and coverage are becoming better, they are still far from what you can do with Java.

I will not talk about modularity, refactoring etc. when building an application with more than 5 developers (sometimes working from different locations) but of course this is also one of the reason I like GWT (and Java). 

So in the end, we picked GWT. I do not regret it, but I am still concerned.
-->

#JSNI

From what I have experienced JSNI is very powerful. I did not encounter any major blocking point using it (I coded some callbacks and some other tricky stuff). I may not have faced all the possible problems but I strongly beleive that with some efforts, you can achieve almost whatever you want, by using JSNI. The only real question is "how much time" is it going to take you. In the end, time is money, so if you are not productive, it will cost you a lot of money. Is it worth the effort ?

The example of one2team highcharts integration is a very good example. In order to be able to use it, we had to redefine all the native javascript chart options into interfaces. Interfaces are mandatory if you want to be able to test easily your client code using standard junit tests. It will make it possible to mock the highcharts options objects implementations using [mockito](https://github.com/mockito/mockito) for example.

Then we had to implement those interfaces using [JavaScriptObject](http://www.gwtproject.org/javadoc/latest/com/google/gwt/core/client/JavaScriptObject.html) (JSO).

This was a tedious work and hard to maintain as you need to constantly keep up to date with the javascript library. So everything that can help here is welcome !


<!-- Example 

{% highlight javascript %}

{% endhighlight %}

{% highlight java %}

public interface Credits {

	Credits setEnabled(boolean b);

}

public class JSMCredits extends JSMBaseObject implements Credits {

	@Override
	public JSMCredits setEnabled(boolean enabled) {
		this.enabled = enabled;
		return this;
	}

	public boolean istEnabled() {
		return enabled;
	}

	private Boolean enabled;

}

{% endhighlight %} -->

# What is coming ? Winter ?

<iframe width="640" height="360" src="//www.youtube.com/embed/wFMD1GXR2Tg" frameborder="0" allowfullscreen></iframe>

#### @JsInterface and @JsProperty

No need to implement JSO anymore !

{% highlight java %}
 @JsInterface
 interface MyJsInterface {
   void action1(String x);
   AnotherObject action2(int x, int y);
   @JsProperty void setX(int x);
   @JsProperty int getX();
   @JsProperty boolean hasX();
   default void someOtherMethod() { /* some java code here */ }
 }
{% endhighlight %}

This will behave like following JSO in current GWT compiler (just for illustration):

{% highlight java %}
 class MyJsObjectImpl extends JavaScriptObject implements MyJsInterface {
   native void action1(String x) /*-{ this.action1(x); }-*/;
   native AnotherObject action2(int x, int y) /*-{ return this.action2(x, y); }-*/;
   native void setX(int x) /*-{ this["x"] = x; }-*/;
   native int getX() /*-{ return this["x"]; }-/;
   native boolean hasX() /*-{ return "x" in this; }-/;
   native void someOtherMethod() /*-{ 
     if ('someOtherMethod' in this) {
       this.someOtherMethod(); 
     } else {
       this.@MyJsObjectImpl::super$someOtherMethod()(); 
     }
   }-/;
   private void super$someOtherMethod() { 
     super.someOtherMethod();
   }
 }
{% endhighlight %}


It works both ways. Meaning that when @JsInterface is implemented by a regular java object, the methods will be exported to object’s prototype and names will be preserved hence make them easily callable from javascript (method names are visible in javascript)


{% highlight java %}
class MyJavaObject implements MyJsInterface {
   @Override
   void action1(String x) { /* some java code here */ }
   @Override
   void setX(int x) { /* some java code here (called via Object.defineProperty) */ }
   ...
   void action3(int y) { /* some java code here (enabled for GWT optimizations) */ }
 }
 {% endhighlight %}


{% highlight javascript %}
 // in javascript assuming obj is an instance of MyJavaObject
 obj.action1("action!!"); // works!
 obj.x = "blah"; // works!
 obj.action3(42); // will not work as action3 is not declared on a @JsInterface
{% endhighlight %}


Goodies : Ray Cromwell implemented the use of fluent interfaces and other nice syntax features that will make our life really easier.

{% highlight java %}
@JsProperty T setX(int x)
{% endhighlight %}

alternate getter/setter syntax, so JavaBean as well as non-Javabean syntax is supported

{% highlight java %}
// JavaBean 
getX() /* getter */
isX() /* getter */
setX() /* setter */
{% endhighlight %}

{% highlight java %}
// non-JavaBean
x() /* getter */
x(arg) /* setter */
{% endhighlight %}

{% highlight java %}
@JsInterface
interface Blah {
  @JsProperty int length();
  @JsProperty void length(int x); // sets length
}
{% endhighlight %}


#### @JsExport

To export a symbol to global scope so that it can be called from javascript. Note that @JsExport can only be applied to constructors, static fields and static methods.


{% highlight java %}
class MyJavaObject implements MyJsInterface {
  @JsExport("my.java.Object")
  MyJavaObject() { ... }
  
  @JsExport("my.java.ObjectPrime")
  MyJavaObject(String name) { ... }
  
  @Override
  void action1(String x) { /* some java code here */ }
  void action3(int y) { /* some java code here (enabled for GWT optimizations) */ }
 }
 {% endhighlight %}

{% highlight javascript %}
  // in javascript:
  var obj = new my.java.Object();
  obj.action1("action!!"); /* works! */

  var obj2 = new my.java.ObjectPrime("Coool!!!");
  obj2.action1("action!!"); /* works! */
  obj2.action3(42); /* will not work as action3 is not declared on a @JsInterface */
{% endhighlight %}


# How does it work ?

As Arnaud Tournier mentionned in the comments below, this javascript code generation process is integrated to the GWT compiler.

I think part of this work was inspired by the GWT generator [GWT-Exporter](https://code.google.com/p/gwt-exporter/) written by Ray Cromwell and others. I think it is a good entry point if you want to understand better how this javscript code generation can work. If you want another example of generator you can have a look to [restyGWT](https://github.com/chirino/resty-gwt), 

I wonder if there will be an impact on compilation time if there are lots of JSO involved ?

# (My) Conclusion

I am really excited to try those new features and I can't wait GWT 3.0 to do so. I think this is going in the right direction. I would like to try those new features in conjonction with GIN and see how things are testable or not. Testability is a very important topic to me (and all TDD fans !). It ensures good quality + non-regression. I will try to find out how being able to use the source code from GWT 3.0 to try those new annotation stuff. 

To do so, I intend to code a new Highcharts wrapper with those new features and publish it to Github. I will give some more details about that in futur articles.

# References

Goktug Gokdogan [A glimpse into the future of GWT (js slideshow)](http://gokdogan.appspot.com/gwtcreate2013/#14)

Ray Cromwell [Gwt.Create Keynote San Francisco (slideshare)](http://www.slideshare.net/cromwellian1/gwtcreate-keynote-san-francisco)

Goktug Gokdogan [Nextgen GWT/javascript Interop (google doc)](https://docs.google.com/document/d/1tir74SB-ZWrs-gQ8w-lOEV3oMY6u6lF2MmNivDEihZ4/edit)