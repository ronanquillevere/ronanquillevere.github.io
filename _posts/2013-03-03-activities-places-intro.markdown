---
layout: post
title:  "GWT Activities and Places framework Introduction"
date:   2013-03-03 20:19:00
Tags: [Activity, Google Web Toolkit, GWT, History, Model, MVP, Place, Presenter, Token, URL, View]

---

Hello everyone,

In the following article I will try to explain and clarify the Google Activities and Places framework.

You can have a look at the following article for more thoughts on nesting activities : [http://wpamm.blogspot.tw/2013/08/nested-activities-alternative-with.html](http://wpamm.blogspot.tw/2013/08/nested-activities-alternative-with.html)

##Model View Presenter (MVP)


For a lot of people, it is not clear wether or not the Activities and Places framework is equivalent to MVP. MVP is a coding pattern. It was pushed by Google in their GWT framework as an improvement of the classical MVC pattern [http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)

###Classical MVC pattern

!![mvc](/img/MVC.png)


###MVP pattern 

!![mvc](/img/MVP.png)


The difference is that the model and the view should not know each other.

The consequence is that the view should be as dumb as possible. No intelligence should be coded in the view. The view is only the graphical representation of the component. All the behaviors should be coded in the presenter. The events are transmitted by the view to the presenter.

This pattern makes it really simple to replace a view component (this can be really useful to test the application with mocked views ...)

Check out the Google I/O 2009 for more explanation on the topic (go to 21m30s)

<iframe width="480" height="360" src="//www.youtube.com/embed/PDuhR18-EdM?t=21m29s" frameborder="0" allowfullscreen></iframe>


Other MVP implementations for GWT have been developed and released before Google released their own in GWT 2.2. Example gwtmvp. [http://www.gwtmpv.org/index.html](http://www.gwtmpv.org/index.html)

##Activities and Places

###Introduction

As stated by the documentation [https://developers.google.com/web-toolkit/doc/latest/DevGuideMvpActivitiesAndPlaces
](https://developers.google.com/web-toolkit/doc/latest/DevGuideMvpActivitiesAndPlaces
)
The Activities and Places framework allows you to create bookmarkable URLs within your application.

So Activities and Places are a lot more than just an MVP framework. Nonetheless this framework has been build around the MVP pattern. The MVP part of this framework is represented by the Activity interface

{% highlight java %}
public interface Activity {

  String mayStop();

  void onCancel();

  void onStop();

  void start(AcceptsOneWidget panel, EventBus eventBus);
}
{% endhighlight %}

Here you find the typical methods you will find in all MVP implementations.
start() : initialize some stuff in a start() like event registrations, add your view to the dom etc.
onStop() : clean some stuff (and memory) like event registrations etc.

Sometimes these methods are called bind() and unbind() in other MVP frameworks.

So an Activity is your Presenter. All your logic should be found there. This is the glue between your view (FlowPanel, AbsolutePanel etc...) and your model (your data).

###Places & history workflow


So now let's take a look a the Activity Place framework in detail. To do so I just used the GWT Activity Place MVP tutorial code found here [http://code.google.com/p/google-web-toolkit/downloads/detail?name=Tutorial-hellomvp-2.1.zip](http://code.google.com/p/google-web-toolkit/downloads/detail?name=Tutorial-hellomvp-2.1.zip). I made it work with eclipse and launched the GWT application.

Then I entered the following url in my brower : http://127.0.0.1:8888/hellomvp.html?gwt.codesvr=127.0.0.1:9997#GoodbyePlace:World!

Here is what happened

!![mvc](/img/ActivityPlaceDiagram.png)

A better version of the diagram focused on the second part

!![mvc](/img/SimpleActivityPlaceDiagram.jpg)

Once the application is launched, the PlaceHistoryHandler get the url token from the Historian. Then the PlaceHistoryHandler calls the PlaceHistoryMapper to get the Place corresponding to the url token. To do so the PlaceHistoryMapper calls the PlaceTokenizer.

Once it has the Place, the PlaceHistoryHandler ask the PlaceController to go to the new Place. It is a kind of redirection. The PlaceController simply sends a PlaceChangedEvent with the new Place wrapped inside.

The ActivityManager listen to this event and asks the ActivityMapper to give back the Activity corresponding to the Place. Then the ActivityManager starts the Activity.

{% highlight java %}
@Override
public void start(AcceptsOneWidget containerWidget, EventBus eventBus)
{
    GoodbyeView goodbyeView = clientFactory.getGoodbyeView();
    goodbyeView.setName(name);
    containerWidget.setWidget(goodbyeView.asWidget());
}
{% endhighlight %}

At the end of the start method, the containerWidget (in this case the whole HTML page) set its widget to the new view.

###Responsibilities


Here are the some of the main objects and their responsibilities

PlaceHistoryMapper : get a Place from a String (url token).
ActivityMapper : get an Activity from a Place.
ActivityManager : starts, stops the activities (it is the Master of Ceremony)

These are really the objects you want a look into if you want to change something in the frameworks like adding parameters to your urls ...

I think I will end there this article as it is long enough. I will probably detail some other things in other articles. Hope it helped !
