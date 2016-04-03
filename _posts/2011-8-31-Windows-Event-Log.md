---
layout: post
title: Reading from the windows event log
---

Dead simple this, you create an instance of System.Diagnostics.EventLog, and you can iterate over the entries of any log you want, provided your account has access. There is an overload to the constructor that will allow you to read the logs of other machines, though I'll admit, I haven't tried it yet!

Anyway...

{% highlight c# %}

//omitting common stuff
using System.Diagnostics;

//have a look in the event viewer to see others
var log = new EventLog("Application");

foreach (EventLogEntry logEntry in log.Entries)
{
    Console.WriteLine(logEntry.Message);
}

{% endhighlight %}

I love when things work out much easier than you think they will :)
