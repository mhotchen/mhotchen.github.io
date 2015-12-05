---
layout: post
title: PHP as a templating language
---

Many templating languages have been built for PHP ([Twig](http://twig.sensiolabs.org/), [Smarty](http://www.smarty.net/), [Dwoo](http://dwoo.org/), etc.). These languages will have a subset of the features of PHP useful for formatting text and some basic programming (looping, counters, basic arithmetic). Here I'll offer some of the more common arguments in favour of templating languages and why I think they're wrong. I'll be using Smarty as a demonstration language because it's the most popular, and the one I have the most experience with.

Front-end developers should never have to deal with PHP
-------------------------------------------------------

This is the most common argument, usually backed by up by saying that it offers the front-end person a cleaner syntax/easier to use language for their uses (I'll get back to this) and that it reduces the amount of opportunities that a front-end person can accidentally break the backend of the code. Firstly, let me demonstrate how a front-end person can use PHP in Smarty, thus negating the supposed risk of them not having access to it.

{% highlight smarty %}
{% raw %}
{php}die('Whoops!');{/php}
{% endraw %}
{% endhighlight %}

Whoops, indeed. If you're using some sort of method for separating your logic from your output (for example, an MVC framework) and you're using OOP then the front-end developer will never have access to the database or any other objects that you don't explicitly pass from the controller to the view, regardless of what language the view uses.

The syntax is cleaner
---------------------

Not always. Compare

{% highlight smarty %}
{% raw %}
{$smarty.now|date_format:'%Y-%m-%d %H:%M:%S'}
{% endraw %}
{% endhighlight %}

With

{% highlight php %}
{% raw %}
<?=date('Y-m-d H:i:s')?>
{% endraw %}
{% endhighlight %}

I know it's a contrived example, but there are plenty like it, and this one came straight from the docs. I started out as a front-end developer and I can't really describe how frustrating Smarty's syntax was for me. No syntax highlighting, a warning in my IDE for every variable or control structure I used, problems with javascript (especially passing variables from PHP to javascript, you have to decide whether to escape the whole javascript block and unescape for the variables like so):

{% highlight javascript %}
{% raw %}
{literal}
	<script type="text/javascript">
		function init() {
			var username = "{/literal}{$username}{literal}";
			// ...
		}
	</script>
{/literal}
{% endraw %}
{% endhighlight %}

Or you used the `{rdelim}` and `{ldelim}` predefined constants to add your braces:

{% highlight javascript %}
{% raw %}
<script type="text/javascript">
	function init() {ldelim}
		var username = "{$username}";
		// ...
	{rdelim}
</script>
{% endraw %}
{% endhighlight %}

There is an option to change the Smarty opening/closing braces to something more sensible, but it uses braces by default, so that's what most people use.

There is one thing I like about Smarty though: because it's a domain specific language it makes it easy to to manipulate string variables, thanks to it's variable modifiers. For example:

{% highlight smarty %}
{% raw %}
{$title|truncate:40:"..."}
{% endraw %}
{% endhighlight %}

Will truncate the string to 40 characters and add a trailing '...' if the title is longer than 40 characters. The (raw) PHP is more complicated:

{% highlight php %}
{% raw %}
<?=(strlen($title) < 40) ? $title : substr(0, 40, $title) . '...'?>
{% endraw %}
{% endhighlight %}

There's also the option to truncate from the middle of the string, or from the boundary of a word, which gets much more complex in PHP.

I don't feel that this justifies an entirely new language though. There are template libraries out there that use PHP as the templating language, and have useful helper functions. I'll demonstrate with `Zend\View` (I especially like the currency helper here).

{% highlight php %}
{% raw %}
<?=$this->currency($value)?>
{% endraw %}
{% endhighlight %}

Which could be € 1.234,56 or £1234.56, etc. depending on locale. With `Zend\View` it's also easy to add your own helpers, which could be things that are specific to the project, so you could create a helper for calculating which posts are popular based on frequency of replies.

Is it really necessary to add yet another language to an already language abundant stack, especially when PHP is so close to getting it right? Wouldn't it be great if your back-end developers could easily read how the front-end developers are using the data given to them without having to scour the Smarty docs, and wouldn't it be nice for the front-end developers to have the auto-completion, PHPDoc for the variables, and be able to read the code that defines the variable so they can understand where it comes from?

I know that I never benefited from templating languages in PHP, and after many years of web development, I'm still waiting for the case where it is beneficial to anyone around me.
