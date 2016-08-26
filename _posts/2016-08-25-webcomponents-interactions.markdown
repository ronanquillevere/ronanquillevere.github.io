---
layout: post
title:  "Web Components interactions"
date:   2016-08-25 23:59:00
Tags: [javascript]
Categories: [javascript]
---

Hello everyone,

# What this article is

I researched quickly how it is possible to interact with web components and I wanted to share my results. The idea behind this quick study was to understand how it would be possible to interact with a web components through various frameworks ([angular](https://angularjs.org/), [GWT](http://www.gwtproject.org/), [extjs](https://www.sencha.com/products/extjs/)), simply because this is the frameworks we use in my company. But before that I needed the basic understanding of web components.

# What this article is not

This is not an introduction to [web components](http://webcomponents.org/). For that I suggest :

<iframe class="center" width="480" height="360" src="//www.youtube.com/embed/XYlgxre_AF4" frameborder="0" allowfullscreen></iframe>

[http://www.youtube.com/watch?v=XYlgxre_AF4](http://www.youtube.com/watch?v=XYlgxre_AF4)

This article will also not discuss wether you should use or not use web components.

Just keep in mind web components are a group of specs

- Custom Elements
- HTML Imports
- Templates
- Shadow DOM


# Interacting with web components

When I interact with a js library the type of interactions I want to do are usually

- Pass information to that library (options, data etc.)
- Being able to be recalled at some point to trigger some actions inside my business logic

If you take the example of [Highcharts.js](http://www.highcharts.com/), you want to pass your data to draw a chart, then you migh want to be recalled when you click on a point to trigger some actions in your app.

So I tried to code a web component with those capabilities

# Result
Here is the result [https://github.com/ronanquillevere/chromeonly-element](https://github.com/ronanquillevere/chromeonly-element)

How it looks (img)
<img class="center" src="https://raw.githubusercontent.com/ronanquillevere/chromeonly-element/master/wc1.png" alt="result">

The `index.html` page

{% highlight html %}
<!doctype html>
<html>
<head>

    <meta charset='utf-8'>
    <title>Web Component test 1</title>
    <link rel='import' href='./mo-wc-test1/mo-wc-test1.html'>

</head>
<body>
    
    <mo-wc-test1 id='wc1'></mo-wc-test1>
    
    <mo-wc-test1 id='wc2'></mo-wc-test1>
    
    <script type='text/javascript'>
        var wc = document.querySelector('#wc1');
        wc.render(
            {
                name : 'hello web component 1',
                onButton1Clicked : function(){
                    alert('wc1 button 1 clicked');
                }
            });

        var wc2 = document.querySelector('#wc2');
        wc2.render(
            {
                name : 'hello web component 2',
                onButton1Clicked : function(){
                    alert('wc2 button 1 clicked');
                }
            });


        setTimeout(function() 
            {
                wc.render(
                {
                    name : 'hello web component 1 - updated',
                    onButton1Clicked : function(){
                        alert('wc1 button 1 clicked - updated');
                    }
                });

            }, 5000);
    </script>
</body>
</html>
{% endhighlight %}

My component creation code

{% highlight html %}
<link rel='import' href='mo-wc-test1_template.html'>

<script>
(function() {
    
    // currentDoc = mo-wc-test1.html document 
    // document = index.html document
    // importedDoc = mo-wc-test1_template.html document

    // Creates an object based in the HTML Element prototype
    var element = Object.create(HTMLElement.prototype);
    console.log('initialize element');

    //Retrieving the current document and not the host (index.html) document
    var currentDoc = document.currentScript.ownerDocument;

    var importedDoc = currentDoc.querySelector('link[rel="import"]').import;

    element.wcOptions = {};

    // Fires when an instance of the element is created
    element.createdCallback = function() 
    {
        // Adding a Shadow DOM
        var rootElement = this.createShadowRoot();
        
        // Adding a template
        var template = importedDoc.querySelector('template');
        var clone = document.importNode(template.content, true);
        rootElement.appendChild(clone);
        
        //add specific component values (not on prototype)
        this.titleZone = rootElement.querySelector('#titleZone');
        
        this.registerButtonsCallbacks(rootElement);

        console.log('element created');
    };

    // Fires when an instance was inserted into the document
    element.attachedCallback = function() 
    { 
        console.log('element attached');
    };

    // Fires when an instance was removed from the document
    element.detachedCallback = function() 
    {
        console.log('element detached');
    };

    element.attributeChangedCallback = function(attr, oldVal, newVal) {};   

    
    element.registerButtonsCallbacks = function(rootElement) 
    { 
        var button1 = rootElement.querySelector('#button1');
        var button2 = rootElement.querySelector('#button2');  
        var self = this;
        
        button1.addEventListener('click', function(){
            element.clickButton1(self);
        });
        
        button2.addEventListener('click', function(){
            element.clickButton2(self);
        });
    };

    element.clickButton1 = function(that) 
    {
        if (that.wcOptions.onButton1Clicked)
            that.wcOptions.onButton1Clicked();
        else
            console.log('button1 was clicked but no callback is set');
    };

    element.clickButton2 = function(that) 
    {
        if (that.wcOptions.onButton2Clicked)
            that.wcOptions.onButton2Clicked();
        else
            console.log('button2 was clicked but no callback is set');
    };

    element.render = function(wcOptions)
    {
        this.wcOptions = wcOptions;
        this.titleZone.textContent = this.wcOptions.name;
        console.log('rendered');
    };
    


    // Registers custom element
    document.registerElement('mo-wc-test1', {
        prototype: element
    });

   
}());
</script>
{% endhighlight %}

The html template used

{% highlight html %}
<template>
    <style>
      #button1 {
        color : red;
      }
      #button2 {
        color : blue;
      }
    </style>
    <div id="container">
        <button id="button1">button1</button>
        <button id="button2">button2</button>
        <span id="titleZone"></span>
    </div>
</template>
{% endhighlight %}

# Some explanations
So what I did, appart from creating a very basic web component, is simply enrich the prototype of my web component so that I can pass data to it and callbacks

This is how I pass data to my web component, first I retrieve it then I call the render method defined on the element prototype

```javascript
var wc = document.querySelector('#wc1');

wc.render(
{
    name : 'hello web component 1',
    onButton1Clicked : function()
    {
        alert('wc1 button 1 clicked');
 	}
});
```            

On the element prototype I have the following method. Note that `this` is the object calling the method so this.wcOptions is set on the web component object and not on the prototype.

```javascript
element.render = function(wcOptions)
{
    this.wcOptions = wcOptions;
    this.titleZone.textContent = this.wcOptions.name;
    console.log('rendered');
};
``` 

For the click button callbacks the only trick is to manage properly how to pss the `this` object. The createdCallback method is called by the web component object, so `this` is the instance calling the method. Because in Javascript method are bound when executed and not when they are declared I need to save and pass the right reference to the this object to the button click listeners.

```javascript
element.registerButtonsCallbacks = function(rootElement) 
{ 
    var button1 = rootElement.querySelector('#button1');
    var button2 = rootElement.querySelector('#button2');  
    var self = this;
    
    button1.addEventListener('click', function(){
        element.clickButton1(self);
    });
    
    button2.addEventListener('click', function(){
        element.clickButton2(self);
    });
};
``` 

Then my clickButton1 prototype method will check if a callback has been defined in the options passed to the render method.

```javascript
element.clickButton1 = function(that) 
{
    if (that.wcOptions.onButton1Clicked)
        that.wcOptions.onButton1Clicked();
    else
        console.log('button1 was clicked but no callback is set');
};
```


Hope this article will help some of you.