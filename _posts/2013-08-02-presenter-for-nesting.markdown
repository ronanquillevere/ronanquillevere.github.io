---
layout: post
title:  "Nested Activities : an alternative with Presenters"
date:   2013-08-02 20:19:00
Tags: [Activity, Google Web Toolkit, GWT, MVP, Nesting, Place, Presenter, Thomas Broyer]
Categories: [gwt,activityplaces]
---

If you are reading this article it means you probably have already read the article called [GWT 2.1 Activities – nesting? YAGNI!](http://blog.ltgt.net/gwt-21-activities-nesting-yagni/)  by [Thomas Broyer](https://plus.google.com/113945685385052458154?rel=author). If you did not, I suggest to have a look at his article.

In this article I will reformulate the idea explained by Thomas Broyer and propose another solution and try to compare them.

##Activity Place Pattern

First you need to understand how the Activities and Places pattern works in detail. Below is a quick overview with the major objects involved.

<img class="center" src="/img/SimpleActivityPlaceDiagram.jpg" alt="mvp" width="700px">

When you go from one Place to another, at some point, the framework will throw a PlaceChangeEvent. Inside this event you will find a Place object corresponding to the URL of the Place your are trying to go to.

This event will be listened by the ActivityManager. The ActivityManager will call the ActivityMapper to give him back the Activity (instance) corresponding to that new Place.

Then the ActivityManager will try to start() this Activity.

So the first time you design an application using the Activities and Places pattern you will probably have something like : one web page = one url = one activity

But if your web page is complex ,with distinct regions inside (a header, a side bar, a content area etc.), with components inside that you might want to reuse in other pages you end up trying to create sub-activities or "nested" activities as Thomas call them.

##One Place - Many Activities


This is how I would describe Thomas' solution. Again, please have a look at his article [GWT 2.1 Activities – nesting? YAGNI!](http://blog.ltgt.net/gwt-21-activities-nesting-yagni/) .

The idea is the following

- 1 ActivityManager + 1 ActivityMapper for each region of a web page
- All ActivityManagers share the same Eventbus
- 1 layout for all the pages of the application. This layout defines all the possible regions
- If a region is not needed on a certain page, the corresponding ActivityMapper will return null and the ActivityManager will not display the region (meaning the region will be hidden)

So let's say you want to go to a new Place in your application. The PlaceChangeEvent will be fired. All ActivityManagers will receive the event ( they all share the same Eventbus ) each of them will call his own ActivityMapper returning a specific Activity.

If you take the example of a "side bar" region (left part of your web page). On a Place called placeA, the ActivityMapper will return an activityA, but on a placeB, the ActivityMapper may return an activityB or may return null thus hiding the "side bar region".


##One Place - One Activity - Many Presenters

Here is my proposal, another way of doing this is the following: You need to introduce another interface which is a kind of Presenter and that is not managed by the Activities and Places framework.

Your Presenter interface should probably have those kind of methods (the presenter interface should probably also have some kind of getView() method).

{% highlight java %}
public interface Presenter {
    void init();
    void dispose();
    //...
}
{% endhighlight %}

They will be initialized/disposed by the Activity when it starts or stops.

An Activity will contain as many presenters as regions in the page (and presenters might contain nested presenters inside them). The Activity can have a view which is a DockLayoutPanel and when the Activity starts, it can add the presenters' views inside the different areas of its DockLayoutPanel view.

<img class="center" src="/img/ActivityPresenter.png" alt="mvp" width="700px">

So the start method of the activity should look like this (you would probably pass the Eventbus to your presenters in the init method but I just want to focus on the idea, not the real implementation).

{% highlight java %}
activity.start(){
    //...
    headerPresenter.init();
    getView().addNorth(headerPresenter.getView());

    sideBarPresenter.init();
    getView().addLeft(sideBarPresenter.getView());

    contentPresenter.init();
    getView().add(contentPresenter.getView());
    //...
}
{% endhighlight %}

The disposal of the presenters should be called when the activity stops.

Of course your Activity can have a simpler layout for its view than the DockLayoutPanel.

So instead of having many ActivityManagers, you can have only one. The activity will be the master of ceremony between your different presenters. And you will have one Activity per Place.

You could use the Activity Interface for your Presenters but I think it is kind of dirty ...

###Pros

- One ActivityManager
- One ActivityMapper
- One Activity per Place

###Cons

- The history can be tricky to handle if you want to change the state of your presenters and update the URL without "really" changing the place
- Creating a new Presenter interface and being sure that presenters are initialized/disposed correctly changing the main layout from place to place

Ok I think I will stop here, I will probably add some details in the futur if it is not understandable enough.

Tell me what you think of it, I am sure my proposal is a bad idea :D. 