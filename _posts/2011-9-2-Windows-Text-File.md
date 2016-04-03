---
layout: post
title: Reading from the windows event log
---

Reading data from a text from a text file is easy, once you know what's happening:

{% highlight c# %}

using (var file = new StreamReader(filePath))
{
    ((line = file.ReadLine()) != null)
    {
        //Do stuff here
    }
}

{% endhighlight %}

What we're doing is getting a memory stream of the file, and reading a line of text as defined by MSDN:

>A line is defined as a sequence of characters followed by a line feed ("\n") or a carriage return immediately followed by a line feed ("\r\n"). The string that is returned does not contain the terminating carriage return or line feed. The returned value is a null reference (**Nothing** in Visual Basic) if the end of the input stream is reached.

 This is the easiest way, but not the safest. If the .ReadLine() method throws an OutOfMemoryException (can't create a buffer for the return string), you loose your place in the stream, and need to re-initialise it (could be a problem in large files with no \n or \r\n chars). You can solve it by pre-allocating a char buffer and populating it with the .Read() method, but then you have to parse for EOL yourself. Ahh well, I'll maybe get round to it next week.
