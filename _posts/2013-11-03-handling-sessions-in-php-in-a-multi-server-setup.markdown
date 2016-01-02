---
layout: post
title: Handling sessions in PHP in a multi-server setup
---

One of the first problems that people typically encounter when scaling a PHP application from a single server to multiple servers is how to handle sessions. PHP defaults to storing sessions in a file on the server, which, again by default, isn't shared with other servers, so if a user starts their session on one server, then their next request is on another server, the second request can't access the session data on the first server, and so it will generate a new session file.

There are 2 camps for solving this, each with their pros and cons: sticky sessions, and sharing the session data:

### Sticky sessions

What this means, is once a user has made a request, you make note of the server that received that request and you keep sending that user's requests to the same server. This can be done by storing the server's name in the user's cookie. There are a couple of ways to achieve this:

1. You could do it based on existing user data, so for example, all user's that have an ip address that end in [0-4] go to backend1, and the remaining go to backend2
2. Alternatively, store the name of the server that received the first request in a cookie (or as part of the session id), then have your proxy read that cookie and if it's set (and valid!) then forward the request to the stored backend

The main advantages of sticky sessions are:

* Easier to achieve
* You're not relying on any additional services, or servers, that could go down

The biggest disadvantages are:

* If one of your servers go down, every user with a session associated with that server loses all their session data (at least until the server is back up)
* Your load balancing becomes less flexible. If one of your servers is being hammered by a few nasty requests, then you can't send the rest of the requests to an alternative server, any user associated with that server will have to accept the slowness

I personally wouldn't use sticky sessions, mainly for the losing sessions if a server goes down reason.

### Sharing session data

Based on my research, this branch of thinking is much more diversified and complex, but when achieved correctly it has many advantages over sticky sessions.

There are ways to share session data across servers without writing any code:

* [Memcached](http://php.net/manual/en/memcached.sessions.php) (PECL lib)
* [MM](http://php.net/manual/en/session.installation.php) (built in, if PHP is built with the `--with-mm` flag)

The benefit of both is their easy setup. With memcached, it's a pecl install then an edit of your php.ini. MM is just a
single line n your php.ini.

The problem with both of these options is that they're in memory. Memcache is, as it's name suggests, for caching, and this means that session data may be purged to make room for other data. With MM, if you do an apache restart, or restart your server, all session data is gone for good. When it comes to not frustrating your users, this is bad.

Custom session handlers
-----------------------

PHP makes it quite easy to build a custom session handler that works seamlessly with your existing sessions: [session\_set\_save\_handler](//php.net/session_set_save_handler). All you need is a session opener, a session closer, a session reader, a session destroyer and a session garbage collector. It's actually surprisingly simple. PHP 5.4 even comes with [a handy interface](//www.php.net/manual/en/class.sessionhandlerinterface.php) and [a concrete implementation](//www.php.net/manual/en/class.sessionhandler.php) to get you started.

Additionally Symfony has [a few bundled session handlers](//symfony.com/doc/current/components/http_foundation/session_configuration.html) ([github source](https://github.com/symfony/symfony/tree/master/src/Symfony/Component/HttpFoundation/Session/Storage/Handler)) for you to either use or to help get you started. As of writing this though, the Symfony session handlers are quite basic, and lack features like session locking, or any kind of resiliency for when your session storage service inevitably goes down.

### Session locking

<img src="/assets/posts/session-override.png" class="inline" alt="Without locking the session can be overwritten" />

One important and often forgotten part of sessions is locking. A user should not be able to have 2 sessions open simultaneously. The reason for this is that if you open a long-running request, then a second request comes along, writes to the session then closes, then the first request writes to the session then closes, you just lost all of the data from the second request. What should happen (and what happens with the default session handler) is when the first request has opened the session file, the second request should wait until the first request has closed the session before it begins working with it.

A naive demonstration of locking:

{% highlight php %}
{% raw %}
<?
// in session open handler
$lockKey = "lock-$sessionId";
while (file_exists("_sessions/$lockKey")) {
    usleep(10);
}

touch("_sessions/$lockKey"); // now we have the lock

// in session close handler, unlock
unlink("_sessions/$lockKey");
?>
{% endraw %}
{% endhighlight %}

<img src="/assets/posts/session-with-locking.png" class="inline" alt="With locking we can see that the session isn't overwritten" />

The above is a less robust version of what `flock` does on the file system which is what PHP relies on under the hood. You would probably want to use something else that's shared across your infrastructure such as memcache or redis for storing your session lock.

You must ensure that no matter what your PHP does, it executes the unlocking or else your user will not be able to open a session on subsequent requests. [register\_shutdown\_function](//php.net/register_shutdown_function) is built for exactly this type of scenario.

PHP by default will wait for a session lock indefinitely. In the example above the PHP script will wait for a long time to get a session lock, far surpassing the max execution time set because `usleep` (and `sleep`) do not count towards the length of time the PHP script has been executing for. Due to the nature of network volatility an eventual exit path and a TTL on the lock are highly recommended to make sure users aren't locked out of their session indefinitely if a script fails to unlock properly.
