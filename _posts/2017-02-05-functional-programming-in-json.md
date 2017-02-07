---
layout: post
title: Functional Programming in JSON.
---

You may know how to do functional programming in Javascript. Maybe LISP. Or maybe even Haskell!

But today let me show you something you've never seen: **Functional programming in pure JSON**.

I will talk about:

1. How LISP creates a program out of lists, and Why JSON is the new "list"
2. How you can actually write a program in JSON
3. How to do functional programming in JSON

#LISP.
It's short for **LIS**t **P**rocesing. It's a programming language 100% based on lists. For example here is a list:

```
(0 1 2 3)
```

But this is also a list:

```
(+ 2 3)
```
In this case, the first token in the list is interpeted as the operator and the rest argument. So here it means "add 2 and 3".

You can also define functions using the built in command:
{% raw %}
```
(defun duplicate (N) (+ N N))
```
{% endraw %}


And finally, this:
{% raw %}
```
(defun factorial (N)
  "Compute the factorial of N."
    (if (= N 1)
          1
              (* N (factorial (- N 1)))))
```
{% endraw %}

LISP was created in the 1950s when the **list** was the most popular form of data structure. Arrays, linked lists, these were all "lists".

Now, almost 60 years later, what is the most popular form of data structure?

# JSON.

Fast forward to 2017, our data mostly lives on the cloud, and the focus is no longer about how you store data locally. More important question is how to store and transmit data remotely. And in 2017, the most popular format to exchange data over the Internet is JSON.

Here's a JSON object:

{% raw %}
```
{
  "items": ["a", "b", "c"]
}
```
{% endraw %}

Even though JSON started its life as an offspring of Javascript (JSON is short for JavaScript Object Notation), it became de facto standard when it comes to data exchange.

So, what if you could do what LISP did, but for JSON? What if it's possible to build a completely functional, production application simply using JSON?

# Procedural Programming in JSON.

That's a part of what [Jasonette](https://www.jasonette.com) does. Along with the ability to express an entire mobile app interface using JSON (Declarative Programming), another important feature is a way to express a sequence of actions using JSON (Procedural Programming). So how does it work?

We start with something called an `action`. An action is a JSON object that represents a single task, such as accessing the camera, playing audio, making network requests, rendering, etc. 

Each "action" consists of up to four attributes:

1. `type`: name of a native function to run
2. `options`: arguments to pass to the function
3. `success`: success callback to run when the function returns success
3. `error`: error callback to run when the function returns error

Using just these four attributes, you can build a fully functional, production app. Here are the building blocks:

<br>

## Arguments

Functions are not so meaningful without arguments. We want a function to perform different actions based on the arguments we pass to it.

Here we pass `title` and `description` attributes to `$util.alert` action to create a custom alert.

{% raw %}
```
{
  "type": "$util.alert",
  "options": {
    "title": "Alert",
    "description": "Hello world"
  }
}
```
{% endraw %}

It displays an alert that looks like this:

If we were to express this in Javascript it would have looked something like this

```js
$util.alert({title: "Alert", description: "Hello world"});
```
* NOTE: Jasonette is not implemented with Javascript, I'm just drawing an analogy here just to make a point

## Callbacks

When it comes to mobile app development, most actions run asynchronously, which means we need some way to represent **callbacks** if we want to execute one action after another.

Let's take another example:

{% raw %}
```
{
  "fetch": {
    "type": "$network.request",
    "options": {
      "url": "https://jsonplaceholder.typicode.com/posts"
    },
    "success": {
      "type": "$render"
    }
  }
}
```
{% endraw %}

This chain of actions makes a network request to the jsonplaceholder server and renders the result using whichever template you define (also in JSON).

Here, the `success` object is the callback. It waits until the `$network.request` completes successfully and then runs.

If we were to express this in Javascript it would look something like this

```js
function fetch(){
  $network.request({url: "https://jsonplaceholder.typicode.com/posts"}, function(res){
    $render(res);
  });
}
```
* NOTE: Again, I'm using this as an analogy to make a point. This is not how Jasonette is implemented internally.

## Inline javascript

We must go deeper. Let's try some crazy stuff using Jasonette's inline Javascript feature.

{% raw %}
```
{
  "fetch": {
    "type": "$network.request",
    "options": {
      "url": "https://.....",
      "data": {
        "guid": "{{var str=$cache.guid; var chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/='; for ( var block, charCode, idx = 0, map = chars, output = ''; str.charAt(idx | 0) || (map = '=', idx % 1); output += map.charAt(63 & block >> 8 - idx % 1 * 8)) { charCode = str.charCodeAt(idx += 3/4); if (charCode > 0xFF) { throw new InvalidCharacterError('btoa failed'); } block = block << 8 | charCode; } return output;}}"
      }
    },
    "success": {
      "type": "$render"
    }
  }
}
```
{% endraw %}

Here's what's going on:

1. We make a network request using the data specified inside `options.data`.
2. In this case the value of `guid` is a template expression, which means it will be evaluated when the action is executed.
3. The template expression takes `$cache.guid` (assuming that there's a `guid` value stored in the cache) and generates a hash.
4. Then it makes the network request, after which it calls `$render` via its `success` callback.


# Problem with Procedural Programming.

Above code works just fine, so why do we need to worry about "functional programming" at all?

Well here are some problems with the `fetch` action:

1. It uses `$cache.guid`, which means it will produce different results depending on what value is stored on the cache at the point of execution.
2. It does a lot of different things, from a network call to hash generation to rendering. This means this code is not reusable in other contexts.

For example, we have this nifty `guid` function which generates a hash value, but we can't reuse it since it's built into this particular action call chain.

How can we extract this out as a standalone function somehow so we can reuse it without any side-effects?

#$lambda.

This is where `$lambda` comes in.

`$lambda` is the latest new addition to Jasonette. It's an action whose sole purpose is to trigger another action. Unlike other actions which actually do something on their own, $lambda only functions as a mediator that

1. Executes another action
2. Waits for it to return
3. Continue executing with the return value

Before we get into `$lambda`, let's take a look at how we can define a self-contained function.

## Defining a function

Here's what it looks like when we extract the `btoa` action out into a standalone function. All it does is compute the hash and return.

{% raw %}
```
{
  "btoa": {
    "type": "$return.success",
    "options": {
      "hashed": "{{var str=$jason.guid; var chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/='; for ( var block, charCode, idx = 0, map = chars, output = ''; str.charAt(idx | 0) || (map = '=', idx % 1); output += map.charAt(63 & block >> 8 - idx % 1 * 8)) { charCode = str.charCodeAt(idx += 3/4); if (charCode > 0xFF) { throw new InvalidCharacterError('btoa failed'); } block = block << 8 | charCode; } return output;}}"
    }
  }
}
```
{% endraw %}

Notice that:

1. We have a new action type called `$return.success`. This is a special purpose action designed for returning the control back to the action that triggered the current action via `$lambda`.
2. the function uses `var str=$jason.guid;` instead of `var str=$cache.guid;`. `$jason` is a special variable reserved for accessing the parameters passed in from the triggering action.

For example, when you trigger a function called `fun` with the following JSON: 


{% raw %}
```
{
  "trigger": "fun",
  "options": {
    "key": "abc"
  }
}
```
{% endraw %}

you can access the `{"key": "abc"}` from inside the `fun` action using `$jason.key`:

{% raw %}
```
{
  "fun": {
    "type": "$util.banner",
    "options": {
      "title": "Display key",
      "description": "{{$jason.key}}"
    }
  }
}
```
{% endraw %}



## Calling the function

Now that we have defined a function, let's call it using `$lambda`.

The `$lambda` action takes two arguments as its `options`:

1. `name`: The name of the function to trigger
2. `options`: An options object to pass to the triggering action.

{% raw %}
```
{
  "fetch": {
    "type": "$lambda",
    "options": {
      "name": "btoa",
      "options": {
        "guid": "{{$cache.guid}}"
      }
    },
    "success": {
      "type": "$network.request",
      "options": {
        "url": "https://.....",
        "data": {
          "guid": "{{$jason.hashed}}"
        }
      },
      "success": {
        "type": "$render"
      }
    }
  }
}
```
{% endraw %}

Notice that:

1. We trigger the `btoa` function we created above and pass arguments using the `options` attribute. The `$cache.guid` stays outside of the **btoa** function this time.
2. Notice how the `$lambda` has a `success` callback. Which means it will wait until `btoa` returns, and then use its return value to move forward. You can see how the `$network.request` call now uses the `$jason.hashed` value.

## Simplifying $lambda with 'trigger'
This `$lambda` action will be one of the most frequently used actions going forward. This is why there's a syntactic sugar for it.

Instead of using the `$lambda` action directly, you can use the `trigger` keyword. For example instead of this:

{% raw %}
```
{
  "type": "$lambda",
  "options": {
    "name": "some_action",
    "options": {
      "key1": "value1",
      "key2": "value2"
    }
  },
  "success": {
    "type": "$render"
  },
  "error": {
    "type": "$util.toast",
    "options": {
      "text": "Error"
    }
  }
}
```
{% endraw %}

You can use: 

{% raw %}
```
{
  "trigger": "some_action",
  "options": {
    "key1": "value1",
    "key2": "value2"
  },
  "success": {
    "type": "$render"
  },
  "error": {
    "type": "$util.toast",
    "options": {
      "text": "Error"
    }
  }
}
```
{% endraw %}

# Conclusion
There are so many cool things about this, but it may not be very clear what benefit this brings in the grand scheme of things. 

Here's why I think this is really cool:

1. Each function is a self-contained entity which when composed with other functions can do all kinds of powerful things.
2. With JASON (the JSON-based language that powers Jasonette), we can express any function in JSON.
3. JSON is the new "List". JSON is ubiquitous.

I will stop here and leave the rest to your imagination. But I am sure some of you are as excited as I am. Please join the community and contribute if so!

![Network](/assets/network.png)
