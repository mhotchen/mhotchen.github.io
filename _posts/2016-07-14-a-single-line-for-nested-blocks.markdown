---
layout: post
title: "A single line for nested blocks"
---

I think all programmers have their own stylistic choices. All programmers working on software read by many people will know to keep their own opinions in check and to adapt a popular formatting scheme for the languages given. When building my own projects for myself, I adapt a slightly different style that improves readability for me. I would never use the following in a professional setting because I know it will throw people off and confuse them, since it's *unusual*.

I've always been a fan of single line `if` statements where appropriate. I most often use them for guard clauses at the beginning of a function/method.

I noticed in scala that I would write something like the following:

{% highlight scala %}
for (c <- chunks; l <- c.lines if !l.isAdded) yield l
{% endhighlight %}

This contains what would be three block level statements in C-like languages (`for`, `if`, `yield`). Technically four if you count the fact that it's effectively a nested loop. I started toying with this idea in C++ and think that it adds some terseness and clarity:

{% highlight cpp %}
if (inside) switch (e->type) {
    case SDL_MOUSEMOTION:
        sprite = BUTTON_SPRITE_MOUSE_OVER_MOTION;
        break;
    case SDL_MOUSEBUTTONDOWN:
        sprite = BUTTON_SPRITE_MOUSE_DOWN;
        break;
    case SDL_MOUSEBUTTONUP:
        sprite = BUTTON_SPRITE_MOUSE_UP;
        break;
}
{% endhighlight %}

This would normally be written as something like the following:

{% highlight cpp %}
if (inside) {
    switch (e->type) {
        case SDL_MOUSEMOTION:
            sprite = BUTTON_SPRITE_MOUSE_OVER_MOTION;
            break;
        case SDL_MOUSEBUTTONDOWN:
            sprite = BUTTON_SPRITE_MOUSE_DOWN;
            break;
        case SDL_MOUSEBUTTONUP:
            sprite = BUTTON_SPRITE_MOUSE_UP;
            break;
    }
}
{% endhighlight %}

In languages with a `for..in` style construct the original scala code can be written as something along the lines of:

{% highlight javascript %}
for (c in chunks) for (l in c.lines) if (!l.isAdded()) lines.push(l);
{% endhighlight %}

Instead of:

{% highlight javascript %}
for (c in chunks) {
    for (l in c.lines) {
        if (!l.isAdded()) {
            lines.push(l);
        }
    }
}
{% endhighlight %}
