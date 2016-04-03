---
layout: post
title: Parsing dates in .NET
---

Had to parse a time stamp today that DateTime.Parse() laughed at. Thought about writing an IFormatProvider (one of the overloads of the .Parse method) which looked complicated, but instead stumbled on this:
{% highlight c# %}
DateTime.ParseExact(String, String, IFormatProvider)
{% endhighlight %}
You know how you can custom output DateTimes based on a string (e.g. 'dd/MMM/yy')? It works in reverse too.

Here's my usage case:

{% highlight c# %}

var timeStamp = DateTime.ParseExact("10/Oct/2000:13:55:36 -0700","dd/MMM/yyyy:HH:mm:ss K", CultureInfo.InvariantCulture)

{% endhighlight %}

I use InvariantCulture because I'm parsing time stamps with a standardised format, so don't need to worry about ':' or '/' separators. Not sure what else it affects, so I'll have to do a bit more research!
