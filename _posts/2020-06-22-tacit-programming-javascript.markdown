---
layout: post
title:  "A light introduction to tacit programming with JavaScript"
date:   2020-06-22 19:52:00
---
Tacit programming is a style of programming in which you don't identify the arguments your functions operate on. Instead, you define your functions by composing other functions.

It's also known as the "point-free" style, and it's a common pattern in functional programming.

The aim of this post is to dig into what that exactly that means, how it's possible in JavaScript and why you might want to code in that style.

Let's look at a simple example for motivation.

Imagine we want to automatically generate an email address for new starters at our company, from their names. Our rule for doing this is that we want to take the person's surname, change it to lowercase, then append "@companyname.com".

Here's how we might do that in code:

{% highlight javascript %}
function getSurname(fullName) {
  let nameParts = fullName.split(" ");
  return nameParts[nameParts.length - 1];
}

function appendDomain(localPart) {
  return `${localPart}@companyname.com`;
}

function getEmailAddressFromName(fullName) {
  return appendDomain(getSurname(fullName).toLowerCase());
}
{% endhighlight %}

Here, the `getEmailAddressFromName` function is really just an amalgamation of 3 other functions, with no additional logic: `getSurname`, `toLowerCase` and `appendDomain`.

To really see this, it would help to redefine `toLowerCase` so that it's just a function rather than a string method:

{% highlight javascript %}
function getSurname(fullName) {
  let nameParts = fullName.split(" ");
  return nameParts[nameParts.length - 1];
}

function toLowerCase(string) {
  return string.toLowerCase();
}

function appendDomain(localPart) {
  return `${localPart}@companyname.com`;
}

function getEmailAddressFromName(fullName) {
  return appendDomain(toLowerCase(getSurname(fullName)));
}
{% endhighlight %}

Now it's easy to see that `getEmailAddress` is just 3 functions applied in sequence.

It would be great if we could declare `getEmailAddress` using something like the imaginary syntax below:

{% highlight javascript %}
let getEmailAddressFromName = appendDomain of toLowerCase of getSurname
{% endhighlight %}

Unfortunately this isn't real JavaScript. But if it was, it would be a clean way of expressing that one function is just a composition of 3 others. This is what we would call a **point-free** definition.

That's a bit of a strange term, but it makes sense when you consider that a "point" in this context means an argument.

Is there some way we could approximate this in JavaScript?

We can definitely try!

Let's make things simpler by considering the case where we want to compose just 2 function together.

Keeping the same example, we might want to define a `getLowerCaseSurname` function to be `getSurname` followed by `toLowerCase`:

{% highlight javascript %}
function getLowerCaseSurname(fullName) {
  return toLowerCase(getSurname(fullName));
}
{% endhighlight %}

Simple enough.

Now let's define a function called `compose` that looks like this:

{% highlight javascript %}
function compose(f, g) {
  return x => f(g(x));
}
{% endhighlight %}

This might be confusing at first glance. What does this function do?

We can see it returns another function. That function takes a single argument, `x`, applies `g` to it, then applies `f` to it. Aha! So `f` and `g` must both be functions.

So we can see that compose takes two functions as arguments and returns another function.

This sounds like what we wanted to do with `getLowerCaseSurname`. What happens if we pass in `toLowerCase` and `getSurname` to compose? It would return the following:

{% highlight javascript %}
x => toLowerCase(getSurname(x))
{% endhighlight %}

Hopefully you can see that is equivalent to our definition of `getLowerCaseSurname` above.

So, actually, we could have written the following:

{% highlight javascript %}
let getLowerCaseSurname = compose(toLowerCase, getSurname);
{% endhighlight %}

This is very clean. And point-free! We've defined `getLowerCaseSurname` purely in terms of other functions without mentioning the data the function operates on.

What if we wanted to apply three or more functions in a row, like with `getEmailAddressFromName`?

We could define a more generic `compose` function that works with a variable number of arguments:

{% highlight javascript %}
function compose(...functions) {
  return x => functions.reduceRight((gx, f) => f(gx), x);
}
{% endhighlight %}

This version is a little harder to understand, so don't worry if it's not clear. What matters is that, using this function, we can define `getEmailAddressFromName` as follows:

{% highlight javascript %}
let getEmailAddressFromName = compose(appendDomain, toLowerCase, getSurname);
{% endhighlight %}

This is really not far from what we envisioned earlier using the imaginary "of" keyword. It's point-free, and very readable: you can easily see that one function has been made by composing several others in sequence.

The `compose` function is essential to tacit programming and functional programming in general. You will find it (sometimes with a different name) in any functional programming library, including [Lodash](https://lodash.com/), [Underscore](https://underscorejs.org/) and my personal favourite, [Ramda](https://ramdajs.com/).

Here's how you would use it in Ramda:

{% highlight javascript %}
const R = require('ramda');

let ceilAbs = R.compose(Math.ceil, Math.abs);

console.log(ceilAbs(-3.7)); // Logs 4
{% endhighlight %}

Ramda also provides a function called `pipe`, which does the same thing as `compose` except that the order of the arguments is reversed:

{% highlight javascript %}
const R = require('ramda');

let ceilAbs = R.pipe(Math.abs, Math.ceil);

console.log(ceilAbs(-3.7)); // Logs 4
{% endhighlight %}

Whether to use `compose` or `pipe` is a matter of preference and may depend on the situation. Sometimes it's more intuitive to to read the list of functions you're composing from left to right, in the order they will be applied. In this case, use `pipe`.

Whether you choose `compose` or `pipe`, these two functions only get you so far in writing point-free code. Without a few more utility functions up your sleeve, you'll quickly encounter a situation that's hard to translate to the point-free style.
Fortunately, Ramda provides many more functions to make tacit programming easier, such as `ifElse`, `cond`, `either`, `both`, and many more.

These are outside the scope of this post, but I encourage you to check out the [Ramda documentation](https://ramdajs.com/docs/#) if you're interested.

Let's look at one more example to hammer home how clean tacit programming can be.

Let's say we have an array of numbers and we want to find the even ones. We could do the following:

{% highlight javascript %}
function getEvenNumbers(numbers) {
    return numbers.filter(x => x % 2 === 0);
}

let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

console.log(getEvenNumbers(numbers));
{% endhighlight %}

Let's try to give `getEvenNumbers` a point-free definition instead.

Here we've used a simple arrow function as our filter condition inside the `getEvenNumbers` function. The arrow function returns true if a number is even, by checking if it's equal to 0 modulo 2.

But expressions featuring the modulus operator aren't the most readable, so let's move this out into a named function:

{% highlight javascript %}
function isEven(number) {
    return number % 2 === 0;
}

function getEvenNumbers(numbers) {
    return numbers.filter(x => isEven(x));
}

let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

console.log(getEvenNumbers(numbers));
{% endhighlight %}

This is definitely more readable. But let's look at our new filter condition more closely. It's now an arrow function that returns the result of calling `isEven` on its argument.

Hmm, ok... an arrow function that just returns the result of another function. Doesn't that seem a bit pointless?

We could have just written the following:

{% highlight javascript %}
function isEven(number) {
    return number % 2 === 0;
}

function getEvenNumbers(numbers) {
    return numbers.filter(isEven);
}

let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

console.log(getEvenNumbers(numbers));
{% endhighlight %}

Here, we pass `isEven` directly into `filter`. This works just fine, of course - `filter` expects its argument to be a function that takes a number and returns a boolean. Often we would use an arrow function here, but `isEven` fits the bill too.

This is cleaner and more readable, and we're getting closer to being point-free. But we have a problem: we call `filter`, which is a method on the variable `numbers`. We can't eliminate our arguments if we have to call methods on them.

Enter Ramda once more. Ramda redefines array methods such as `filter`, `map` and `reduce` to be standalone functions instead. We can use Ramda's version of filter instead:

{% highlight javascript %}
const R = require('ramda');

function isEven(number) {
    return number % 2 === 0;
}

function getEvenNumbers(numbers) {
    return R.filter(isEven, numbers);
}

let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

console.log(getEvenNumbers(numbers));
{% endhighlight %}

This is still not point free, but we can make it so due to another trick Ramda employs: **currying**.

All Ramda functions, including `filter`, are curried by default. If you haven't come across currying before, think of it as a more flexible way of defining functions of multiple arguments, allowing you to provide only some of the arguments at a time.

In the case of `filter`, it means the following two ways of calling the function are equivalent:

{% highlight javascript %}
R.filter(isEven, numbers);
R.filter(isEven)(number);
{% endhighlight %}

In the first line, we've provided both arguments at once, as normal. In the second line, we've called the argument with one argument, then called the result with the second argument. This works just fine for Ramda functions.

The reason this works is that, by calling the function with just one argument, you return a new function that takes the second argument and then applies both arguments to the original function.

If the single-argument version of filter was a separate function, it would be defined something like this:

{% highlight javascript %}
function filterOneArg(arg1) {
    return arg2 => R.filter(arg1, arg2);
}
{% endhighlight %}

The upshot of all of this is that we could define `getEvenNumbers` as follows:

{% highlight javascript %}
let getEvenNumbers = numbers => R.filter(isEven)(numbers);
{% endhighlight %}

But now we can see we no longer need the arrow function at all, which leads us to our point-free holy grail:

{% highlight javascript %}
let getEvenNumbers = R.filter(isEven);
{% endhighlight %}

Hurrah!

Tacit programming and currying are two of the core concepts of functional programming. If you've found this post interesting and want to learn more about functional programming without having to learn a whole new language, I suggest [Professor Frisby's Mostly Adequate Guide to Functional Programming](https://mostly-adequate.gitbooks.io/mostly-adequate-guide/), which introduces core FP concepts from a JavaScript perspective.