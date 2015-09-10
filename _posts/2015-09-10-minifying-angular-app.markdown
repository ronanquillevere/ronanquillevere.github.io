---
layout: post
title:  "Minifying an angular app" 
date:   2015-09-10 23:59:00
Tags: [angularjs, minify]
Categories: [angularjs]
---

Hello,



This is something you will probably have to look at by the end of your project : 

- minifying js and css files (replacing method names by initials, remove line breaks and comments etc…)
- merging js and css files (having only one file for each)

To do so I have used the [minify-maven-plugin](https://github.com/samaxes/minify-maven-plugin). 

In my project I had to run this plugin after the [sass compilation](https://github.com/darrinholst/sass-java/tree/master/sass-java-maven) (need the css files to be there to compact them !)

Under the hood it uses 2 things :
- [YUI Compressor](http://yui.github.io/yuicompressor/)
- [Google Closure Compiler](https://developers.google.com/closure/compiler/?hl=en)

After executing the plugin I end up with a file called `app.min.js` and `style.min.css` that each contain all the js minified and all the css minified !

#Angular impact

In order to be able to minify your angular code you have to declare your controller, factories etc in a certain manner. The goal is to avoid your injected dependencies to be renamed by the minifier process.

See this [note on minification](https://docs.angularjs.org/tutorial/step_05#a-note-on-minification)

To sum up you have to declare your controller like this

{% highlight javascript %}
phonecatApp.controller('PhoneListCtrl', ['$scope', '$http', function($scope, $http) {...}]);
{% endhighlight %}

#HTML pages

In order to use `app.min.js` and `style.min.css` I had to change the index.html page of our application to reference those new files. Problem is that those 2 files are generated inside the `target/APPNAME-VERSION` directory. Thus they are not usable when using `mvn tomcat:run`. I could have done some ugly hacks in the pom to copy those file in the src directory (that is used by tomcat when running `mvn tomcat:run`) but this is really too ugly.

The solution I came up with is simply to have 2 index pages 
one page for production : `index.html`
one page for dev : `indexdev.html`

Inside the `indexdev.html` I reference all the js files and css files. It enables debug. The only thing is to remember to type the right url in the browser when starting the application in dev mode.

For production I believe you can exclude the `indexdev.html` when generating your war in maven (not tried yet).

Index.html

{% highlight html %}
<!-- *** Production css *** -->
<link rel="stylesheet" href="assets/css/style.min.css">

<!-- Js dependencies -->
<script src="assets/libs/angular/angular.min.js"></script>
<script src="assets/libs/angular-ui-router/release/angular-ui-router.min.js"></script>
<script src="assets/libs/jquery/dist/jquery.min.js"></script>
<script src="assets/libs/bootstrap/dist/js/bootstrap.min.js"></script>

<!-- *** Production js *** -->
<script src="app/app.min.js"></script>

{% endhighlight %}


Indexdev.html


{% highlight html %}
<!-- *** Production css *** -->
<!-- <link rel="stylesheet" href="assets/css/style.css"> -->

<!-- *** Debug css *** -->
<link rel="stylesheet" href="assets/css/mycss1.css">
<link rel="stylesheet" href="assets/css/mycss2.css">
<link rel="stylesheet" href="assets/css/mo-bootstrap.css">


<!-- Js dependencies -->
<script src="assets/libs/angular/angular.min.js"></script>
<script src="assets/libs/angular-ui-router/release/angular-ui-router.min.js"></script>
<script src="assets/libs/jquery/dist/jquery.min.js"></script>
<script src="assets/libs/bootstrap/dist/js/bootstrap.min.js"></script>


<!-- *** Production js *** -->
<!-- <script src="app/app.min.js"></script> -->

<!-- *** Debug js *** -->
<script src="app/aa1.app.routes.js"></script>
<script src="app/aa2.app.module.js"></script>
<script src="app/shared/header/headerDirective.js"></script>
<script src="app/components/home/homeController.js"></script>
<script src="app/components/about/aboutController.js"></script>
<script src="app/shared/clients/helloClient.js"></script>

{% endhighlight %}

#POM

Here is an extract of my pom.xml

{% highlight xml %}
<plugin>
   <groupId>com.samaxes.maven</groupId>
    <artifactId>minify-maven-plugin</artifactId>
    <version>${minify-maven-plugin.version}</version>
    <executions>
      <execution>
        <id>default-minify</id>
        <goals>
         <goal>minify</goal>
        </goals>
        <!-- To be able to use it in mvn tomcat:run, will run after sass 
          compile -->
        <phase>compile</phase>
        <!-- <phase>package</phase> -->
        <configuration>
          <charset>UTF-8</charset>
          <cssSourceDir>assets/css</cssSourceDir>
          <cssFinalFile>style.css</cssFinalFile>
          <jsSourceDir>app</jsSourceDir>
          <jsFinalFile>app.js</jsFinalFile>
          <jsEngine>CLOSURE</jsEngine>
          <closureLanguage>ECMASCRIPT5</closureLanguage>
          <closureAngularPass>true</closureAngularPass>
          <cssSourceIncludes>
            <cssSourceInclude>**/*.css</cssSourceInclude>
          </cssSourceIncludes>
          <jsSourceIncludes>
            <jsSourceInclude>**/*.js</jsSourceInclude>
          </jsSourceIncludes>
        </configuration>
      </execution>
    </executions>
  </plugin>

{% endhighlight %}

Hope it will help you !



