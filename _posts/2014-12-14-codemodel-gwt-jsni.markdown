---
layout: post
title:  "Generating JSNI code for GWT with codemodel"
date:   2014-12-14 23:59:00
Tags: [GWT,  JSNI, Codemodel, code]
Categories: [GWT]
---

Hi everyone, 

In the past 4 months, outside my work time, I have been writing a new GWT wrapper for the great [highcharts](http://www.highcharts.com) library. It is not finished yet, but I have now something that is usable I think. If you are interested have a look the the [Github project page](https://github.com/highcharts4gwt). If you are not familiar with JSNI please have a look to the [corresponding GWT project page](http://www.gwtproject.org/doc/latest/DevGuideCodingBasicsJSNI.html).

So the idea was to be able to generate the wrapper code, I wanted to write a wrapper that I would not have to maintain by hand everytime a new option was made available by the highcharts team. The highcharts library use a Json file that describes all the available options, it is available [here](http://api.highcharts.com/highcharts), just click the "as JSON" to see the raw file.

One option typically looks like this :

{% highlight javascript %}
{
	values: null,
	title: "animation",
	extending: "",
	excluding: "",
	isParent: false,
	since: "2.3.0",
	demo: "",
	deprecated: false,
	seeAlso: "",
	defaults: "true",
	fullname: "tooltip.animation",
	name: "tooltip--animation",
	parent: "tooltip",
	returnType: "Boolean",
	description: "Enable or disable animation of the tooltip. In slow legacy IE browsers the animation is disabled by default."
}
{% endhighlight %}

So my idea was to read that option file and to generate the wapper for GWT. This is what I am going to talk about in the following article.

##Architecture

Before going into the details of the code generation I would like to explain quickly the architecture I decided to adopt. Below are the classes I want to generate using Codemodel. One interface and two implementations. The "Jso" (JavaScriptObject) implementation will extends the GWT JavascriptObject class. This is where we will code the JSNI methods to wrapp highcharts javascript calls.

The "Mock" is a pure Java implementation that can be used for Unit testing. No need for GWTTestCase, we will just need to inject the mock implementation instead of the JSO one in the context of a test. So that if our wrapper is used inside a presenter, you can still test your presenter in a pure Junit test.

<img class="center" src="/img/JsoMockArchi.png" alt="JsoMockArchi" width="400px">


##Codemodel

A good place to start for using codemodel would be on that [blog](http://blogtech.cardosi.net/2013/03/19/tutorial-java-codemodel-basics/) where you will find various very good article to start. I use the following codemodel revision.

{% highlight xml %}
<dependency>
	<groupId>org.glassfish.jaxb</groupId>
	<artifactId>codemodel</artifactId>
	<version>2.2.10-b140310.1920</version>
</dependency>
{% endhighlight %}


My point is not to rephrase what you will find in the blog mentionned before but more to answer this specific question : "how do I generate a native method ?"

##JSNI

This is what a JSNI method looks like

{% highlight java %}
public static native void alert(String msg) /*-{
  $wnd.alert(msg);
}-*/;
{% endhighlight %}

Codemodel is intented to generate Java code, there is no way codemodel will be able to generate the "/\*-" and the "-\*/" or the content of the method which is pure Javascript. So I needed to find some hack to do it anyway.

I found out that in codemodel, I could use "throw" part to insert my native code. The only drawback is that all my native method have to throw a RuntimeException. This is acceptable as this is an [unchecked exception](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html). So I created a "hack" class to do it.

{% highlight java %}
public class NativeContentHack extends JClass
{

    private final String nativeContent;

    public NativeContentHack(JCodeModel codeModel, String nativeContent)
    {
        super(codeModel);
        this.nativeContent = nativeContent;
    }

    @Override
    public String name()
    {
        return "RuntimeException " + nativeContent;
    }

    @Override
    public JPackage _package()
    {
        return owner()._package("");
    }

    @Override
    public JClass _extends()
    {
        return null;
    }

    @Override
    public Iterator<JClass> _implements()
    {
        return null;
    }

    @Override
    public boolean isInterface()
    {
        return false;
    }

    @Override
    public boolean isAbstract()
    {
        return false;
    }

    @Override
    protected JClass substituteParams(JTypeVar[] variables, List<JClass> bindings)
    {
        return null;
    }

    @Override
    public String fullName()
    {
        return "RuntimeException " + nativeContent;
    }

}
{% endhighlight %}


And this is how you can use it


{% highlight java %}
NativeContentHack nativeCode = new NativeContentHack(jCodeModel, "/*-{$wnd.alert(msg);}-*/");
jDefinedClass
	.method(JMod.NATIVE + JMod.FINAL + JMod.PUBLIC, void.class, "alert")
	._throws(nativeCode)
	.param(String.class, "msg");
{% endhighlight %}


This will give you 

{% highlight java %}
public final native void alert(String msg) throws RuntimeException /*-{
  $wnd.alert(msg);
}-*/;
{% endhighlight %}



