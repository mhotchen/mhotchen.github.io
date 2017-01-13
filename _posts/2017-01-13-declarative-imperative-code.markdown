---
layout: post
title: Declarative imperative code
---

Declarative programming, in very basic terms, is declaring what you want a program to deliver without necessarily explaining how the program should achieve that. There are many examples of declarative languages and DSLs, with the two most famous being SQL and LINQ.

There are some lessons to be learned from declarative programming which can be applied to an imperative program. In this article I want to discuss a couple and how they can improve readability, composability, and safety within PHP applications.

## Lesson one: variables

Declarative languages of course have variables, but more often than in imperative style programming they become unnecessary because you're not worrying about state or transformation, you're simply declaring what you want and it's up to the underlying system to work out how to get there.

In imperative programming we don't have luxuries like letting the compiler plan how to build your results for you, but we can take away from this that variables aren't as important as they initially seem. In fact, I would posit that for some languages, declaring variables is an entirely unnecessary endeavor. Let's take a look at one example in PHP:

{% highlight php %}
<?php

class Format
{
    public function applyFormat(array $words, \Closure $format): array
    {
        $formattedWords = [];
        foreach ($words as $word) {
            $formattedWords[] = $format($word);
        }

        return $formattedWords;
    }
}

$transform = function (string $word): string {
    $returnedWord = "";
    $length = strlen($word);
    for ($i = 0; $i < $length; $i++) {
        // I'm being purposefully obscure here, just for fun :)
        $returnedWord .= chr(ord($word[$i]) ^ 32);
    }
    return $returnedWord;
};

$format = new Format;
$words = ['Hello', 'wORLD'];
$results = $format->applyFormat($words, $transform);

$expected = ['hELLO', 'World'];
var_dump($expected == $results);

// bool(true)
{% endhighlight %}

In the above code there's a class Format with the method applyFormat on it. That takes a list of words and a closure, applying the closure to the words and returning the results. We declare a closure $transform, which converts the letters in a word to the opposite case. Finally it's all put together and the results are tested.

Let's say that the interface for Format needs to stay the same but we're free to refactor the code how we see fit. Let's take another stab at this but this time we won't use any variables:

{% highlight php %}
<?php

class Format
{
    public function applyFormat(array $words, \Closure $format): array
    {   
        return array_map($format, $words);
    }   
}

function invertCase(string $letter): string
{
    // I'm being purposefully obscure here, just for fun :)
    return chr(ord($letter) ^ 32);
}

var_dump(
    (new Format)->applyFormat(
        ['Hello', 'wORLD'],
        function (string $word): string {
            return implode(array_map('invertCase', str_split($word)));
        }
    )
    ==
    ['hELLO', 'World']
);

// bool(true)
{% endhighlight %}

Oh, nice! Completely variable free. There is no state, no parameters are modified (actually there weren't any modified in the original as well) and because there's no variables each piece becomes smaller. Each piece is much more declarative in nature as well so there's no code saying how to build the new array or how to loop through the letters (I mean the map does both of those things, but the implementation details are masked). If str_split didn't already exist then it could also be written as follows:

{% highlight php %}
<?php

function fake_str_split(string $word, int $splitLength = 1): array
{
    return strlen($word) <= $splitLength
        ? [$word]
        : array_merge(
            [substr($word, 0, $splitLength)],
            fake_str_split(substr($word, $splitLength), $splitLength)
        );
}

var_dump(fake_str_split("hello") == ['h', 'e', 'l', 'l', 'o']);

// bool(true)

var_dump(fake_str_split("world", 2) == ['wo', 'rl', 'd']);

// bool(true)
{% endhighlight %}

This also follows the [divide and conquer](https://en.wikipedia.org/wiki/Divide_and_conquer_algorithm) pattern, a popular pattern amongst those artisanal functional programmer types. Although this is obviously a slow, and very memory intensive solution for PHP, it's almost always easier to start out with no variables/state and add them intentionality where bottlenecks occur.

So recapping this first lesson, what are some benefits we've seen?

* By not declaring variables there is no state or modification to worry about
* Because you can't declare variables, each part needs broken down to a much more granular level promoting safety through isolation and code reuse
* Some of the implementation details become hidden from sight, either through your own more granular parts, or through the use of more declarative approaches that are built in to the language

## Lesson two: map/reduce/filter

By now everyone and their mother is using the map/reduce/filter patterns on their arrays. This reduces state which helps when dealing with concurrency as well as helping the readability when doing non-trivial transformations.

In addition to the functional style benefits that these patterns bring, there are also declarative improvements as well. You're no longer concerned about how the underlying platform builds your new data, you're only concerned with what you want the data to look like.

Because PHP's syntax for map/filter/reduce is quite verbose, let's make a very quick wrapper to reduce the overhead for the coming examples:

{% highlight php %}
<?php

class Arr
{
    private $arr;

    public function __construct(array $arr)
    {   
        $this->arr = $arr;
    }   

    public function map(\Closure $closure): Arr 
    {   
        return new Arr(array_map($closure, $this->arr));
    }   

    public function filter(\Closure $closure): Arr 
    {   
        return new Arr(array_filter($this->arr, $closure));
    }   

    public function reduce(\Closure $closure, $initial = null)
    {   
        return array_reduce($this->arr, $closure, $initial);
    }   
}

function arr(): Arr 
{
    return new Arr(func_get_args());
}

{% endhighlight %}

The above can be extended to include an implementation of the Traversable interface, and implement the ArrayAccess interface to allow looping and adding/removing items to the array add range functionality, summing, etc., but for my simple demonstrations it will suffice in giving us an example of what declarative array access would look like.

Let's say you need to filter an array to find its even numbers, double them, and find the sum. Given an imperative style you might build the following:

{% highlight php %}
<?php

$arr = range(1, 10);
$total = 0;
foreach ($arr as $i) {
    if ($i % 2 != 0) {
        continue;
    }

    $total += $i * $i;
}
var_dump($total);

// int(220)
{% endhighlight %}

Within the loop you have to keep an eye on the $total, $arr, and $i variables to see how things are progressing. Now using the class above it can be rewritten as such:

{% highlight php %}
<?php

var_dump(
    arr(1, 2, 3, 4, 5, 6, 7, 8, 9, 10) 
        ->filter(function (int $i) { return $i % 2 == 0; })
        ->map(function (int $i) { return $i * $i; })
        ->reduce(function (int $sum, int $i) { return $sum + $i; }, 0)
);

// int(220)
{% endhighlight %}

At each point you're not concerned about how you got there, only with what you want the new data to look like. This becomes much more obvious with anything remotely complicated like JSON to object transformations:

First the object:

{% highlight php %}
<?php

class Person
{
    public $name, $age;

    public function __construct(string $name, int $age)
    {   
        $this->name = $name;
        $this->age = $age;
    }   
}
{% endhighlight %}

Imperative:

{% highlight php %}
<?php

$json = [ 
    ['name' => 'Matt', 'age' => 28],
    ['name' => 'Emma', 'age' => 52],
    ['name' => 'Dave', 'age' => 35],
    ['name' => 'Adam', 'age' => 17],
    ['name' => 'Rose', 'age' => 15],
    ['name' => 'Fred', 'age' => 28],
];

$text = "Over 18:";
foreach ($json as $personArr) {
    if ($personArr['age'] <= 18) {
        continue;
    }   

    $person = new Person($personArr['name'], $personArr['age']);
    $text .= " $person->name";
}

var_dump($text)

// string(28) "Over 18: Matt Emma Dave Fred"
{% endhighlight %}

Declarative:

{%highlight php %}
<?php
var_dump(
    arr(
        ['name' => 'Matt', 'age' => 28],
        ['name' => 'Emma', 'age' => 52],
        ['name' => 'Dave', 'age' => 35],
        ['name' => 'Adam', 'age' => 17],
        ['name' => 'Rose', 'age' => 15],
        ['name' => 'Fred', 'age' => 28] 
    )   
    ->filter(function (array $person) {
        return $person['age'] > 18;
    })

    ->map(function (array $person) {
        return new Person($person['name'], $person['age']);
    })

    ->reduce(
        function (string $text, Person $person) {
            return "$text $person->name";
        },
        "Over 18:"
    )
);

// string(28) "Over 18: Matt Emma Dave Fred"
{% endhighlight %}

Ignore for a moment that the Person object is entirely superfluous in this code. There could be writes to the database or something else that is improved by having actual objects in between the request and response (in this case the reduce would be the response).

The difference becomes stark when you want to add additional filters or mappings. Or if you want to refactor it to&mdash;for example&mdash;map before filtering. Manipulating the code is much easier because each part is isolated and simply declaring what it wants the value to look like. You'll also find it easier to compose the sum from smaller parts, for example adding several filter and map functions to your library that can be reused.

Changing the output becomes much easier too. The reduce function can be replaced in isolation if you wanted to reduce it back to JSON for example, whereas in the original code you would be changing variables right through from start to finish.

Also, because of the declarative nature, each part simply says what it wants the data to look like. The filter says "I want only people over 18" and the map says "I want the data in this shape". There's nothing else involved and nothing else to worry about.

So for lesson two, what are the main benefits of declarative programming with arrays?

* With imperative style programming, managing state whilst working with arrays quickly becomes hairy. In a declarative style each part can be isolated from the previous
* With a declarative style each part simply describes what it wants the data to look like, which makes the code easier to read and transform
* Adding more work to the declarative style doesn't detract from readability

Unlike C# (LINQ), Scala (the for construct), and other imperative languages which have built in features for declarative programming, PHP is lacking decent support for this style. But nevertheless these two lessons can be applied today with decent results.
