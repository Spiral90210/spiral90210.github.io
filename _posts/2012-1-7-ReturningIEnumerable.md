---
layout: post
title: Returning an IEnumerable
---


So, you have a data provider, and it has a method signature
{% highlight c# %}
public IEnumerable<string> GetStrings()
{% endhighlight %}

and this does some fancy logic for some reason. In the very common case that you need to return nothing, what's the best way to do it? This code is perfectly valid:
{% highlight c# %}
return null;
{% endhighlight %}

You will get no compile time errors, no warnings and everything will build fine. But what happens if you later do this?
{% highlight c# %}
foreach (var dataObject in provider.GetStrings()) {...}
{% endhighlight %}
You will get a null reference exception. Not what you want I'm sure! Of course you could (and should!) always check that this is not null in the code that uses the provider (flexible in what you accept, rigid in what you return blah blah), but for private methods that you have control over, or incase you forget to add this safety code, there are better ways.

1.
{% highlight c# %}
return new List<string>(0);
{% endhighlight %}
Returns an empty list, which itself implements IEnumerable. You can use any type that implements IEnumerable, like Array. Pretty common knowledge, but for some reason, I prefer number 2.

2.
{% highlight c# %}
return Enumerable.Empty<string>();
{% endhighlight %}

From MSDN:

>Returns an empty IEnumerable<T> that has the specified type argument.

Which really, is exactly what you want. This wont fail in foreach() or lambda expressions (source cannot be null aaragh!!!!), and has the added advantage that it tells you what it is doing in a very readable way.

Using this on your own methods can reduce exceptions thrown where there's no protection for the data returned, but as good practice, anything you don't have control over should be checked thoroughly!
