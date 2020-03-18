---
layout: post
title:  "JavaScript promises"
date:   2020-03-18 08:35:00
---
Promises in JavaScript are an intuitive way to perform asynchronous tasks.

**A promise represents some future value that is not yet known.** Where we are unable to immediately return a value from a function, we can return a promise instead.

By returning a promise, we don’t block execution of the next portion of code. We assign functions to a promise that will run once the promise eventually gives a value.


## A real-world scenario

A real-world scenario that you could model with promises is a Kickstarter campaign.

You want to fund the creation of an album by your favourite artist, so you pledge money towards it. But you don’t receive a copy of the album straight away; it will take the artist time to create the album after they have received the funding, and there’s always a chance they won’t complete it at all.

Instead, you could say that the artist gives you a _promise_ in exchange for your pledge.

The promise is given immediately in exchange for your pledge. The promise is _fulfilled_ later, when you receive the album, or _broken_, if the project fails.

And crucially, you don’t sit around waiting for the album. Once you’ve got your promise, you get on with your life. It’s only when the promise is fulfilled or rejected that you have to think about it again.


## Promise syntax in JavaScript

You can create a new promise in JavaScript using the ```Promise``` constructor.


```
let myPromise = new Promise(...);
```


The ```Promise``` constructor takes a single parameter: a function, known as the executor function of the promise. This function should carry out the long-running task. It takes two functions called ```resolve``` and ```reject``` as parameters, which should be called when the long-running task succeeds or fails, respectively.


```
let myPromise = new Promise((resolve, reject) => {
    // Perform long-running task
    ...
    // Resolve
    resolve(result);
});
```


To do something with the value passed into the resolve function (the result of the long-running task), you can use the ```then``` method to attach a handler to the promise.


```
myPromise.then((result) => {
    // Do something with the result of the long-running task
    console.log(result);
});
```


Any handlers attached using the then method will be called once the ```resolve``` function is called from within the executor function of the promise.

However, the long-running task in the executor function may fail, and you may call ```reject``` in that function instead. To do something with the error passed into the ```reject``` function, attach a handler using the ```catch``` method.


```
myPromise.catch((error) => {
    // Do something with the error returned by the long-running task
    console.log(error);
});
```



### An example

Let’s take the Kickstarter example from above and turn it into code.


```
function makeAlbum() {
   return new Promise((resolve, reject) => {
       setTimeout(() => {
           if(Math.random() >= 0.5) {
               resolve({
                   title: "The Promised Land",
                   length: "50:00"
               });
           }
           else {
               reject("Didn't finish the album");
           }
       }, 1000);
   });
}

function pledge() {
   console.log("I'm pledging to support the album.");
   return makeAlbum();
}

function goToWork() {
   console.log("I'm going to work.");
}

function goOnHoliday() {
   console.log("I'm going on holiday.")
}

pledge().then((album) => {
   console.log(`I've got the album. It's called ${album.title}.`);
}).catch((error) => {
   console.log("I'm sad they didn't finish the album.");
});
goToWork();
goOnHoliday();
```


The ```makeAlbum``` function returns a promise; its executor is a long-running task that can succeed or fail. Here we’ve modeled album creation by setting a timer for 1 second that randomly succeeds or fails.

The ```pledge``` function simply logs some text and then returns the same promise we get from ```makeAlbum```.

We define some other functions (```goToWork``` and ```goOnHoliday```) that represent other tasks we have.

We then call ```pledge```, followed by these two other functions. But we attach some handlers to the result of ```pledge```.

Remember, these handlers won’t be run immediately. They will fire only when ```makeAlbum``` completes.

When you run the code, the output should look like this half the time:


```
I've pledged to support the album.
I'm going to work.
I'm going on holiday.
I'm listening to the album!
```


And this half the time:


```
I've pledged to support the album.
I'm going to work.
I'm going on holiday.
I'm sad they didn't finish the album.
```


When ```pledge``` is called, ```makeAlbum``` starts running asynchronously. While it is still running, ```goToWork``` and ```goOnHoliday``` are called. Then, finally, when ```makeAlbum``` finishes, one of the attached handlers is called, resulting in the last line of output.


## Promise chaining

Sometimes you need to create a promise based on the result of another promise. This is known as _promise chaining_.

To extend our Kickstarter example, let’s imagine you want to review the album once you’ve got it. But before you can review it, you need to listen to it, which takes time.

We define two new functions, ```listenToAlbum``` and ```reviewAlbum```.


```
function listenToAlbum(album) {
   return new Promise((resolve, reject) => {
       console.log("I'm listening to the album!");
       setTimeout(() => {
           resolve(Math.round(Math.random() * 10));
       }, 1000);
   })
}

function reviewAlbum(rating) {
   console.log(`I give the album a score of ${rating}/10.`);
}
```


Then, we chain them to the previous promises.


```
pledge().then((album) => {
   console.log(`I've got the album. It's called ${album.title}.`);
   return listenToAlbum(album);
}).then((rating) => {
   reviewAlbum(rating);
}).catch((error) => {
   console.log("I'm sad they didn't finish the album.");
});
goToWork();
goOnHoliday();
```


Notice that the first handler we attached returns a promise, to which we can attach a second handler.


## Async/await

Promises in JavaScript are powerful, but writing the code for them can get tiresome, especially if you’re dealing with large promise chains.

You can simplify things by using _async/await_ syntax instead. With async/await, you can write asynchronous code that looks much more like traditional synchronous code.

This syntax can only be used inside functions. The function where you’re using it must be declared as an async function by using the ```async``` keyword.


```
async function myFunc() {
    ....
}
```


Once a function has been declared async, you can make use of the ```await``` keyword within it.


```
let result = await promise;
doSomething(result);
```


Used in front of a promise, the ```await``` keyword will return the result of the promise, _once it has been fulfilled_. Execution of that portion of code is halted while the executor function of the promise runs, and resumes once it has finished.

This is non-blocking; the JavaScript engine can process other code in the meantime.

This is equivalent to attaching a handler to the promise:


```
promise.then((result) => {
    doSomething(result);
});
```


The two code snippets do exactly the same thing, but the async/await syntax is a little simpler. The benefit is more clear when doing promise chaining.

Compare the following snippets:


```
promise.then((result) => {
    return doSomething(result);
}).then(result) => {
    doSomethingElse(result);
};
```

```
let result1 = await promise;
let result2 = await doSomething(result);
doSomethingElse(result2);
```



### An example

We can rewrite part of our Kickstarter example using async/await syntax.


```
(async () => {
   let album = await pledge();
   console.log(`I've got the album. It's called ${album.title}.`);
   let rating = await listenToAlbum(album);
   reviewAlbum(rating);
})();
goToWork();
goOnHoliday();
```


This is a bit cleaner, but notice that we’ve had to wrap some of the code in a function to declare it async.

This code also doesn’t handle the case where our promise fails.

When using the ```await``` keyword, if a promise fails then it throws an error instead of returning a result. The surrounding code should therefore deal with the error using try/catch syntax.


```
(async () => {
   try {
       let album = await pledge();
       console.log(`I've got the album. It's called ${album.title}.`);
       let rating = await listenToAlbum(album);
       reviewAlbum(rating);
   } catch(err) {
       console.log("I'm sad they didn't finish the album.");
   }


})();
goToWork();
goOnHoliday();
```


<!-- Docs to Markdown version 1.0β19 -->
