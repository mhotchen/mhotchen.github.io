---
layout: post
title: "Faster file navigation in vim"
---

When using vim most newbies will struggle to work with multiple files in a directory, and experienced users will typically use a tree navigation plugin like NERDTree. In modern times I've all but stopped using directory trees since most modern editors include convenient shortcuts for searching for and opening files quickly. For example when using IntelliJ on Ubuntu you can type `<ctrl>+<shift>+N` to bring up a search box for searching for files in the project. Vim does in fact support this sort of file navigation without the need for plugins.

## Include nested directories in the path

The first step is to include nested directories in the path:

{% highlight VimL %}
set path=$PWD/**
{% endhighlight %}

Add the above to your .vimrc. What it does is if you start vim at the root of the following directory structure:

```
src/
`-- game/
  `-- Game.cpp
  `-- Game.h
  `-- Enemy.cpp
  `-- Enemy.h
`-- levels/
  `-- Level01.cpp
  `-- Level01.h
  `-- Level02.cpp
  `-- Level02.h
```

## The :find command

The find command will search the path for the given file name. With the above change this means it will search the entire directory from which vim was opened. As of vim 7.3 you can tab through the matching files, so to cycle through the levels in the above structure:

{% highlight VimL %}
:find Lev<tab>
{% endhighlight %}

```
  Level01.cpp  Level01.h  Level02.cpp  Level02.h
```

An additional feature of the :find command is that it allows fuzzy matching using the asterisk (\*) character:

{% highlight VimL %}
:find *01<tab>
{% endhighlight %}

```
  Level01.cpp  Level01.h
```

Most modern editors have something similar to the above fuzzy matching set to a convenient shortcut, As demonstrated by the IntelliJ shortcut at the beginning of this article. Vim allows you to map any key combination to a shortcut of your preference very easily. I have the following three mappings set up:

{% highlight VimL %}
map <Leader>f :find *
map <Leader>s :sfind *
map <Leader>v :vert sfind *
{% endhighlight %}

My \<Leader\> key is the comma key (By default it's backslash \\. You can change this by adding `let mapleader = ","` to your .vimrc), so I can type `,f01<tab>` to cycle through Level01.cpp and Level01.h. `:sfind` splits the current buffer horizontally with whatever file I open, and `:vert sfind` does the same, but vertically.
