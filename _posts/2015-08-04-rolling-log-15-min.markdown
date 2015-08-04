---
layout: post
title:  "Rolling logs every 15 mins with logback" 
date:   2015-08-04 23:59:00
Tags: [logback]
Categories: [logging]
---

Want to create rolling logs every 15 mins with logback ?

Idea was adapted from Andreas Kruthoff post, [original post](http://mailman.qos.ch/pipermail/logback-user/2012-April/003099.html)

#Create a new appender

This is a fifteen minutes appender but it is easy to modify ... You have to understand that the appender is called only when something is logged. So if nothing is logged for 10 minutes your appender will not be called for 10 minutes.
Usually on real projects this is not an issue as a tons of logs are generated every minute...

So the idea is just to call the rollover method only when a period of 15min has been reached.

{% highlight java %}
public class FifteenMinuteAppender<E> extends RollingFileAppender<E>
{
   private static long start = System.currentTimeMillis(); // minutes
   private int rollOverTimeInMinutes = 15;

   @Override
   public void rollover()
   {
      long currentTime = System.currentTimeMillis();
      int maxIntervalSinceLastLoggingInMillis = rollOverTimeInMinutes * 60 * 1000;

      if ((currentTime - start) >= maxIntervalSinceLastLoggingInMillis)
      {
         super.rollover();
         start = System.currentTimeMillis();
      }
   }
}
{% endhighlight %}

#Setup logback

Use your new appender in your logback configuration file.

{% highlight xml %}
 <appender class="com.your.package.FifteenMinuteAppender" name="YOUR_APPENDER">
    <file>yourfile.csv</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- daily rollover -->
      <fileNamePattern>yourfile-%d{yyyyMMdd-HHmm}.csv</fileNamePattern>
    </rollingPolicy>
    <encoder>
      <pattern>%d{YYYYMMDD-HHmmSS-ZZ};${HOSTNAME};%thread;%msg%n</pattern>
    </encoder>
  </appender>
{% endhighlight %}

