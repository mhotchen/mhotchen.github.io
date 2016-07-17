---
layout: post
title: "A single line for nested blocks"
---

I think all programmers have their own stylistic choices. All programmers working on software read by other people will know to keep their own opinions in check and to adapt a popular formatting scheme for the languages given. So with that in mind I don't recommend the following as more than food for thought, or possibly as something to adapt for personal projects.

When learning scala I picked up on some stylistic choices in that language that make it valuable to often have multiple block-level statements on a single like:

{% highlight scala %}
for (c <- chunks; l <- c.lines) l match {
  case LineAdded(str)   => add(str)
  case LineRemoved(str) => remove(str)
  case ContextLine(str) => context(str)
}
{% endhighlight %}

This contains what would be a nested loop and a switch in C-like languages. I started toying with this idea in C++ and think that it adds some terseness and clarity. Here's a real world example:

{% highlight cpp %}
if (e.type == SDL_KEYDOWN && e.key.repeat == 0) switch (e.key.keysym.sym) {
    case SDLK_UP:    velY -= VELOCITY; break;
    case SDLK_DOWN:  velY += VELOCITY; break;
    case SDLK_LEFT:  velX -= VELOCITY; break;
    case SDLK_RIGHT: velX += VELOCITY; break;
}
else if (e.type == SDL_KEYUP && e.key.repeat == 0) switch (e.key.keysym.sym) {
    case SDLK_UP:    velY += VELOCITY; break;
    case SDLK_DOWN:  velY -= VELOCITY; break;
    case SDLK_LEFT:  velX += VELOCITY; break;
    case SDLK_RIGHT: velX -= VELOCITY; break;
}
{% endhighlight %}

This would normally be written as something like the following:

{% highlight cpp %}
if (e.type == SDL_KEYDOWN && e.key.repeat == 0) {
    switch (e.key.keysym.sym) {
        case SDLK_UP:    velY -= VELOCITY; break;
        case SDLK_DOWN:  velY += VELOCITY; break;
        case SDLK_LEFT:  velX -= VELOCITY; break;
        case SDLK_RIGHT: velX += VELOCITY; break;
    }
}
else if (e.type == SDL_KEYUP && e.key.repeat == 0) {
    switch (e.key.keysym.sym) {
        case SDLK_UP:    velY += VELOCITY; break;
        case SDLK_DOWN:  velY -= VELOCITY; break;
        case SDLK_LEFT:  velX += VELOCITY; break;
        case SDLK_RIGHT: velX -= VELOCITY; break;
    }
}
{% endhighlight %}

In languages with a `for..in` style construct the original scala code can be written as something along the lines of:

{% highlight javascript %}
for (c in chunks) for (l in c.lines) switch l.type {
    case l.ADDED:   add(l.string); break;
    case l.REMOVED: remove(l.string); break;
    case l.CONTEXT: context(l.string); break;
}
{% endhighlight %}

Instead of:

{% highlight javascript %}
for (c in chunks) {
    for (l in c.lines) {
        switch (l.type) {
            case l.ADDED:   add(l.string); break;
            case l.REMOVED: remove(l.string); break;
            case l.CONTEXT: context(l.string); break;
        }
    }
}
{% endhighlight %}

The difference is farly trivial in the grand scheme of things, and I would actually recommend *not* using it on projects worked on by more than one developer. For my individual tastes however this fairly novel nested block formatting improves readability and clarity of intent.
