---
layout: post
title:  "The beauty of pipelining in F#"
date:   2020-06-06 15:39:00
---
A common pattern in functional programming is to chain several functions together in a single expression.

The output of one function becomes the input of the next, and so on, forming a pipeline.

Imagine we want to take a list of numbers, filter out the odd ones, square each one and then take the sum, before printing the result.

This is what it might look like in JavaScript:

{% highlight javascript %}
let filterOdd = x => x.filter(x => x % 2 === 0);
let squareAll = x => x.map(x => x ** 2);
let sum = x => x.reduce((x, y) => x + y, 0);

let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

console.log(sum(squareAll(filterOdd(numbers))));
{% endhighlight %}

This is fine, but the last line contains several nested function calls, and therefore a lot of brackets. You also need to read it from right to left to work out what's happening, which isn't very intuitive for what is essentially a sequence. This example is simple enough, but readability decreases rapidly with more nested function calls.

Let's take a look at how we might implement this in F# instead.

Firstly, a note on syntax. In functional languages like F#, it's common for function application to work without brackets, by listing the arguments after the function name, separated by spaces. For example:

{% highlight fsharp %}
let x = max 3 5
{% endhighlight %}

This line of F# code calls the ```max``` function with two arguments, 3 and 5.

You still need brackets to specify precedence for nested function calls however. For example:

{% highlight fsharp %}
let x = max (max 1 3) 5
{% endhighlight %}

If we reimplement the JavaScript code from above in F#, we end up with something like the following:

{% highlight fsharp %}
let filterOdd = List.filter (fun n -> n % 2 = 0)
let squareAll = List.map (fun n -> n * n)
let sum = List.sum

let numbers = [1 .. 10];

printfn "%d" (sum (squareAll (filterOdd numbers)))
let x = max (max 1 3) 5
{% endhighlight %}

This has the same problems as the JavaScript code: there are many brackets and you need to read from right to left.

Enter the pipeline operator.

The pipeline operator is defined as follows:

{% highlight fsharp %}
let (|>) x f = f x
{% endhighlight %}

Note: operators in F# are typically called using infix notation, so you would typically type ```x |> f``` rather than ```(|>) x f```.

This looks unassuming, but reversing the order in which a function and its argument appear means you can write function chains that read from left to right, and don't need brackets to specify precedence.

This is what our previous example would look like using the pipeline operator:

{% highlight fsharp %}
let filterOdd = List.filter (fun n -> n % 2 = 0)
let squareAll = List.map (fun n -> n * n)
let sum = List.sum

let numbers = [1 .. 10];

numbers |> filterOdd |> squareAll |> sum |> printfn "%d"
{% endhighlight %}

This is much cleaner and easier to understand. You can imagine the value (```numbers```) "flowing" through the chain of functions, from left to right, one after the other. In my opinion, it feels very natural.

The pipeline operator is used heavily in idiomatic F#, which makes F# on the whole very readable. Functional programming languages have a reputation for being hard to read, but this is one case in which I definitely disagree.