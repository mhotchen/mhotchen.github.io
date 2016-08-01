---
layout: post
title: "An experiment in line numbers"
---

In text editors/IDEs line numbers feel like an obvious and intuitive system that can't really be improved upon. Starting from the number 1 each line in the file is given a new number which is a single increment from the previous line. This seems pretty flawless; you wouldn't increment by two, or start in the middle.

The first alternative I saw was in vim. It has a feature called [relative line numbers](http://vimdoc.sourceforge.net/htmldoc/options.html#%27relativenumber%27), which will tell you how far away all the other lines in the file are from the current cursor position. This is very powerful in vim because a typical vim workflow will involve repeating commands over lines (for example to delete several lines at once, or for jumping ahead/back when navigating) but this isn't very helpful in situations outside of vim or vim-like editors. This doesn't completely replace traditional line numbers but rather acts to compliment them. You can see the actual line you're on in the status bar, and you can still navigate to an exact line number if given one (:83 to go to line 83 for example).

The purpose of this article is to show a new numbering system I "invented" over the weekend. I don't think this is a revolution, but it could be a helpful tool for quickly gaining information about a source code file. I've called this new line numbering system "list line numbers".

# "List line numbers"

For each level of indentation, a new line number set is given for that level, separated by a period/dot from the previous indentation's number. For example given the following code:

{% highlight python %}
def fizzbuzz(sequence):
    values = [ 
        [3, "Fizz"],
        [5, "Buzz"]
    ]   
    for i in sequence:
        output = ""
        for test in values:
            if i % test[0] == 0:
                output += test[1]

        if output == "": 
            output = i 

        print(output)

fizzbuzz(range(1, 101))
{% endhighlight %}

The first line would be labeled "1", the next line, because it's indented, would be labeled "1.1", and again because the next line is indented it would go further in being set to "1.1.1", and because the next line is of the same indentation it would become "1.1.2". Below is a complete example:

{% highlight python %}
1         | def fizzbuzz(sequence):
1.1       |     values = [
1.1.1     |         [3, "Fizz"],
1.1.2     |         [5, "Buzz"]
1.2       |     ]   
1.3       |     for i in sequence:
1.3.1     |         output = ""
1.3.2     |         for test in values:
1.3.2.1   |             if i % test[0] == 0:
1.3.2.1.1 |                 output += test[1]
1.3.2.1.2 | 
1.3.3     |         if output == "":
1.3.3.1   |             output = i
1.3.3.2   | 
1.3.4     |         print(output)
1.3.5     | 
2         | fizzbuzz(range(1, 101))
{% endhighlight %}

As you can see it's lengthier than traditional line numbers, but it does provide some information that traditional line numbers wouldn't at a glance:

1. It becomes easier to see where the complexity lies (around 1.3.2-1.3.3), just from the line numbers. On larger files, without even having to read the source you can see where refactoring effort should be focused.
2. It promotes a flatter structure (which compliments the first exercise in [object calisthenics](https://www.cs.helsinki.fi/u/luontola/tdd-2009/ext/ObjectCalisthenics.pdf)).

So that's it. By using indentation to pack a little more information in to line numbers you can get a quick visual cue of what the state of a source file is. [Here's a rough implementation](https://github.com/mhotchen/line-steps) written in C++.
