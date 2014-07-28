---
layout: post
title:  "GIN compiling debug tips"
date:   2014-07-28 23:59:00
Tags: [GWT,  GIN, debug, injection]
Categories: [GWT]
---

Wow ! I have not posted an article for a long time. To be honest I thought that I would have more time to post / code personnal stuff during summer but it is actualy the opposite ! 

All right I have also taken 3 weeks off in May (it is good to be french sometimes), it did not help. Stop talking about my life. Let's talk about GIN and debugging its awful error messages.

This will be a (very) short article focusing on solutions to solve a code not compiling inside the GWT compiler when using GIN. When using injection it is sometimes hard to understand why the gwt code does not compile. Here are some few tips.

* Launch dev mode, then refresh the page. On the second loading, I have no idea why, the error messages are sometime better.

* Check for <> generics not compatible with gwt compiling in java 1.6 mode.

* Be careful with imports. Sometime you import a wrong version of a lib not compatible with gwt ( like guava)

* Last but not least. If possible, get the class that is not compiling and simply try to do a GWT.create of that class inside your module entry before starting injection. It will give better error message.

Hope it will help !