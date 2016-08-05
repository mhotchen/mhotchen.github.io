---
layout: post
title: "Restrictive programming"
---

Restrictive programming is a technique where you limit your usage of certain features of the language you're using. There are also [restricted programming languages](http://c2.com/cgi/wiki?RestrictedProgrammingLanguage) which restrict the features in the language for you.

Restrictions are often helpful in both the design and implementation of programs. Certain features can be immensely powerful but both intent and clarity can be lost through their usage. Let's take a look at two common examples and a third that I've been imposing on myself recently.

## go to

`go to` allows you to place a label somewhere in the code then jump to that label from anywhere else. You can use this for cleaning up after errors, creating loops, and creating alternative paths based on conditionals. This is very common practice in assembly where the only control instruction is `JMP` which allows you to jump around the instruction set to specified labels. In higher level languages there are built in keywords and syntax for handling the most common situations where `go to` is useful without the need for labels and `go to` statements (for example loops with break and continue, and exceptions).

{% highlight c %}
#include <stdio.h>

int main() {
	int i = 0;
loop:
	fprintf(stdout, "%d\n", i++);
	if (i < 10) {
		goto loop;
	}

	return 0;
}
{% endhighlight %}
<small>An example of recreating a loop in C</small>

There area two excellent papers written on the use of `go to` in higher level languages: a short letter by Dijkstra titled [go to statement considered harmful](http://www.cs.utexas.edu/users/EWD/ewd02xx/EWD215.PDF) and later a more in depth analysis by Donald Knuth titled [Structured Programming with go to Statements](http://sbel.wisc.edu/Courses/ME964/Literature/knuthProgramming1974.pdf). Neither advocate for the complete elimination of `go to` but suggest that newer languages should implement features that cover the common uses. Knuth specifically advocates the measured use of `go to` where appropriate. The largest C codebase available to me—Linux—does indeed use `go to`, primarily for cleaning up after errors/undesirable flows. By limiting their use of `go to` to specific situations where all alternatives fail to improve legibility, the C kernel has kept a codebase that's reasonable easy to read and has limited bugs given its size and complexity.

## variables

A very high level language I've been using over the past few months is Scala. This language has instilled some practices in me that I wouldn't have picked up alone. One of these habits surrounds the use of variables which are a feature of practically all programming languages. Scala gives you the option of defining both an imperative style variable with the `var` keyword and a functional style variable with the `val` keyword. An imperative variable has the ability to change value over the course of a program's lifetime, whereas with a functional style variable, once the value is set it can't change. This allows for [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency) by guaranteeing that data can't be changed by outside forces. This guarantee makes it easier for programmers to reason about the system as a whole and also helps reduce the chances or certain types of bugs. By using constant variables more liberally in other languages I'm restricting what changes can by made, but I'm making it easier to programmers to reason about my system by giving them certain guarantees.

{% highlight c %}
#include <stdio.h>

int main() {
	const char *pattern = "%d\n";
	const int  count = 10;
	for (int i = 0; i < count; ++i) {
		fprintf(stdout, pattern, i);
	}

	return 0;
}
{% endhighlight %}
<small>An example of using constants to give programmers guarantees in C. Note that [C constants don't guarantee that the data won't change](http://yarchive.net/comp/const.html) but it will help catch certain types of programmer mistakes, and more clearly defines intent.</small>

## register

The final C specific example I want to give is the rarely used `register` keyword. This keyword is used as a *hint* to suggest that a certain variable should be stored in the CPU register. These days the compiler is smart enough to work that out for you so it isn't really worth pursuing this sort of optimization unless you're working under specific limitations where these things are important. However the `register` keyword gives you a couple of restrictions with the variable you're creating:

1. Because you're telling the compiler you want to put it in a CPU register and not in memory, you can't create a reference to it (so no pointers to this variable's memory address)
2. It can't be used in external declarations

So we now have some restrictions that help us reason about the program.

{% highlight c %}
#include <stdio.h>

int main() {
	const char *pattern = "%d\n";
	const int  count = 10;
	for (register int i = 0; i < count; ++i) {
		int *reference = &i;
		fprintf(stdout, pattern, *reference);
	}

	return 0;
}
{% endhighlight %}

<pre>
$ g++ -x c -std=c11 test.c 
test.c: In function ‘main’:
test.c:7:3: error: address of register variable ‘i’ requested
   int *reference = &i;
   ^
</pre>

<small>Note that you *must* use a C compiler. C++ ignores the register keyword so this code will compile fine there.</small>

By using the `register` keyword not as a hint to the compiler, but as a restriction within the code, you're giving yourself and other programmers guarantees that they won't leave any hanging pointers or mess with another part of the system when changing the variable `register` was applied to.

## Restrict to understand

By restricting your use of powerful features it becomes easier to understand your code and gives other programmers (and yourself) more confidence in making changes without introducing bugs. I think all languages could do with more restrictive keywords whilst still giving the ability to use more powerful features when necessary. Other examples I could think of would be guaranteeing tail recursion, certain asymptotic complexity, the number of possible outcomes of a function, or guaranteeing that a function is referentially transparent in languages that don't have that guarantee. By restricting your code to have these guarantees you would be able to give other programmers more freedom to use it and modify it, as well as express more of your intention.
