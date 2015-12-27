---
layout: post
title: A brief history of dynamic web data
---

Although people think Microsoft has been holding the future of the web back with it's poor support for the web standards, they did get one thing right, [XMLHttpRequest](http://en.wikipedia.org/wiki/XMLHttpRequest), which has apparently spawned an entirely new version of the web (Web 2.0). Before Microsoft revolutionized communication between server and browser, how did people achieve the same level of communication between the user's browser and the webmaster's server that we take for granted today? Well, the web was built by hackers, and there are some clever ones out there who faced this problem long before Microsoft came up with their solution.

### Use images

Javascript introduced an Image object in 1996. With this object you can set the image source and the browser will request the image from that source.

{% highlight javascript %}
var image = new Image;
image.src = "http://example.com/request"; // new request is made
{% endhighlight %}

A limitation for this was that the data returned needed to be a valid image, and up until recently (with the advent of canvas), javascript was unable to read the raw image data. This means that you can't return any meaningful data within the image that can be read. This meant that all communication to the server was one way, which simply wouldn't do, so hacks were found:

The first, easiest to use approach was to actually return an image of text, which you could then embed in the page for the user to see. This works well if you're after something simple like showing their name after they're logged in, but isn't ideal when the data is more complex, nor is it friendly to the user.

A second approach was to make the call with the image, then the server will set a cookie with the data to return, which can then be read from javascript. One problem with this method was it's reliance on a cookie, something that users were wary of back in 1996.

Another alternative was to create and return an actual image from the server and use it's dimensions to determine what to output to the user. For example if the user has tried to post a comment you could do something like:

{% highlight javascript %}
var SUCCESS                = 1,
    ERROR_POST_DELETED     = 2,
    ERROR_INVALID_PASSWORD = 3;

var requestImage = new Image;

// ...snip...

if (requestImage.width == SUCCESS) showUserComment(comment);
else showCommentPostError(requestImage.width);
{% endhighlight %}

The maximum width or height of a GIF image is 65,535 (the width and height are stored in the header of the file as 16-bit binary numbers, with the largest 16-bit binary number being 65,535). If it wasn't for this limitation, then it might have been possible to store much more data in the width of a GIF, because it compresses really well (thanks to it's use of LZW compression). In fact, a GIF that's 60,000px wide, 1px tall and the same colour throughout is only 370 bytes.

### With the rise of iframes came the downfall of image hacks

By the time the new millennium had come around, frames and iframes were in popular use. These had mostly solved the issues that people were trying to fix with the image hacks above, specifically you no longer needed to reload the entire page to update a particular area of the website. Javascript could still be used to change the location of a frame, and you could now read the data within the frame. This lead to developers using hidden frames to submit and request data to and from the server. They could then read the data that was returned in the frame and determine what to do with that information.

This was a much bigger leap than the XMLHttpRequest in terms of functionality because it introduced the ability for the server to actually return data. All XMLHttpRequest did was give a concrete, cross browser implementation of something that developers had been hacking in to their code since the late 90's.

### On to the present

Now we want push data from the server to the browser. There's an official standards for this, and implementations of WebSockets/Server-sent events already exist in most modern browsers (currently by Chrome, Firefox and IE10), but until people are no longer using IE9 or less, and until Opera has an up-to-date implementation, we're stuck using hacks. The most popular current techniques are referred to as Comet techniques:

#### Ajax long polling

With this technique you would use javascript to make an AJAX request, with a really high timeout, then the server can keep that connection open and respond to it if anything needs pushed to the user. This technique isn't wise when using the popular Apache/PHP combo because not only will Apache dedicate a worker to the request for it's entire duration (which could be minutes if you're building something like a chat application), but there will also be a PHP process running for the duration of that request too.

#### Flash xmlsocket

This is a little trickier to implement, but has a huge performance benefit. The gist of it is that it opens a TCP socket connection with the server, rather than a HTTP request, so you can use something lighter and more dedicated to the role to respond to the user (ie. not a web server). This method is popular with Flash based games and chat systems, because there isn't much in the way of overhead that you don't create yourself, and you can use anything from a Java applet to node.js to pick up the connection and respond from the server side, with a single instance running and listening to the port. The main downfall of this is that many server providers block most ports for security, so unless you're running a dedicated server or virtual server then you'll need to check that additional ports are actually available.

### The future

The future is beginning to look rosy for push notifications. With implementations of WebSockets and Server-sent events already existing, it shouldn't be long before the general population is using browsers that support these technologies. Extremely feature rich, native-like applications are definitely on their way.

What I would like to see now is a standard for TCP and UDP connections from the browser without the use of plugins. In combination with WebGL for GPU rendering, you could have real-time rendering from a particle simulator on the other side of the planet with no need to install any software other than a web browser. Or better yet, counter-strike in your browser :)
