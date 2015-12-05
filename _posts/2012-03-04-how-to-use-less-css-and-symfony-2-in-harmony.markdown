---
layout: post
status: publish
---

*NB: This article is likely outdated. It was written for Symfony 2.0*

Assetic, the assets manager used by Symfony (and a really great tool which is heavily influenced by Python's [webassets](//elsdoerfer.name/files/docs/webassets/)</a> library) has built in functionality for passing your files through a filter in order to generate new output. Using it with LESS files is a snap, as it already comes with 2 filters for generating CSS files from LESS files. One filter works with less.js (the official compiler) through node.js. Another method, common when you're only able to work with PHP, is to use [lessphp](//leafo.net/lessphp/).

Using the official compiler
---------------------------

Of course, this is the recommended method. If you have root access to your server it's also very easy to set up.

### Install node.js, npm and less

[Install node.js](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager). The latest version of node.js now comes with npm already, but run `npm` after installation to make sure. If it doesn't exist, then install npm:

{% highlight bash %}
$ curl http://npmjs.org/install.sh | sh
{% endhighlight %}

Now install the less compiler:

{% highlight bash %}
$ npm install less
{% endhighlight %}

Now you need to find where your node.js binary and modules are installed. Here's where they were for me (Arch Linux and Debian):

Binary: `/usr/local/bin/node`

Modules: `/usr/local/lib/node_modules/`

I've seen the modules in `/usr/local/lib/node/` before, though it's possible this is an outdated practice.

### Configure it's use in your project

Open up your project's configuration file in <code>app/config/config.yml</code> and add the following:

{% highlight yaml %}
assetic:
    filters:
        cssrewrite: ~
        less:
            node: /usr/local/bin/node
            node_paths: [/usr/local/lib/node_modules]
            apply_to: "\.less$"
{% endhighlight %}

The `apply_to` parameter isn't essential, but it does mean that you don't need to apply the filter manually, it'll
automatically apply it to any files ending in `.less`.

Now in your templates all you need to do is:

#### Twig:

{% highlight jinja %}
{% raw %}
{% stylesheets 'path/to/files.less' %}
  <link rel="stylesheet" href="{{ assets_url }}" />
{% endstylesheets %}
{% endraw %}
{% endhighlight %}

#### PHP:

{% highlight php %}
<?foreach ($view['assetic']->stylesheets('path/to/files.less') as $url):?>
  <link rel="stylesheet" href="<?=$view->escape($url)?>" />
<?endforeach?>
{% endhighlight %}

Using the lessphp compiler
--------------------------

In the root of your project you have a `deps` file. Add the following to it:

{% highlight ini %}
[lessphp]
    git=https://github.com/leafo/lessphp.git
    target=/lessphp
    version=v0.3.3
{% endhighlight %}

From the command line run:

{% highlight bash %}
$ bin/vendors install
{% endhighlight %}

Open up your project's configuration file in `app/config/config.yml` and add the following:

{% highlight yaml %}
assetic:
    filters:
        cssrewrite: ~
        lessphp:
            file: %kernel.root_dir%/../vendor/lessphp/lessc.inc.php
            apply_to: "\.less$"
{% endhighlight %}

The `apply_to` parameter isn't essential, but it does mean that you don't need to apply the filter manually, it'll automatically apply it to any files ending in `.less`.

Now in your templates all you need to do is:

#### Twig:

{% highlight jinja %}
{% raw %}
{% stylesheets 'path/to/files.less' %}
  <link rel="stylesheet" href="{{ assets_url }}" />
{% endstylesheets %}
{% endraw %}
{% endhighlight %}

#### PHP:

{% highlight php %}
<?foreach ($view['assetic']->stylesheets('path/to/files.less') as $url):?>
  <?link rel="stylesheet" href="<?=$view->escape($url)?>" />
<?endforeach?>
{% endhighlight %}
