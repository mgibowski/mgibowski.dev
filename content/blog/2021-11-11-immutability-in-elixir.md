+++
title = "Immutability in Elixir"
description = "The beauty of immutability in Elixir."
[taxonomies]
tags = [ "Elixir" ]
+++

Data in Elixir is immutable. It's not immutable by default. It's simply immutable. There's no way around it. This fact has some interesting consequences.

## Benefits of immutability

### Easier reasoning

Immutability saves you from a lot of unnecessary complexity. Your mind doesn't need to maintain a graph of objects with their states. You can focus on fragments in isolation. You are in control. Things are predictible. There is no way that a function you are calling will change the state of a variable you are passing to it. You can rely on that and safely use that variable afterwards.

And when dealing with concurrency, immutability truly is a game-changer.

### Easier concurrency

Let's revise some wording. _Parallelism_ means two or more threads of execution running at the same time. _Concurrency_ though has a slightly different meaning. It means that there are two or more logical threads of execution that may run in parallel, and need to collaborate between each other. In my native language, polish, the literal translation of concurrency would be _"konkurencja"_. If you translate it back to english, the word you get is _"competition"_. I think it describes concurrency pretty well, as in many cases it's all about various threads of execution competing for some common resources.

If you ever wrote concurrent code in Java or C++, you know that it's not simple at all. It takes a lot of effort to avoid race conditions and deadlocks. In order to achieve transactionality, you need to apply mechanisms like locks and semaphores with great amount of care and attention. There are only few, very simple primitives that can be mutated atomically (like `AtomicInt`).

Now, let's take a look at Elixir. Here's an example from the book _"Elixir in Action"_ by Saša Jurić:

```ex
 #TODO
```


This function does A,B,C.

You see? No semaphore, no lock, nothing. Just immutable data and it's perfectly thread-safe.

## Immutability in other languages

You can use immutable data in all mainstream languages. However it comes with two major caveats.

First one is that in most languages immutability is optional. Even if you fully commit to it, you will eventually need to deal with mutable state as you use other's people code, for example in libraries.

Second caveat is that operating uniquely on immutable data is often cumbersome.

Let's take a look at an example from Java:

```java
const offers = fetchOffersForId(42);
const offersWithMetadata = addMetadata(offers);
const processedOffers = process(offersWithMetadata);
```

You are forced to give a new name to the same variable (`offers`), just because the concept you are referring to in your code, at each line is at the different point in time.

You may get away from the need of declaring those intermediate `const` values by using a fluent api style of programming. However, that is not a perfect solution either. First of all, it requires very specific design of API, which may not be the case for the code you need to invoke. Secondly, it may deteriorate your debugging experience, as the 1:1 relationship between the number of line and method invocation, which is so helpful, is no longer there.

## The way we do it in Elixir

Elixir brings much better solutions to the table.

Firstly, the pipeline operator. Which actually is a macro.

```elixir
fetch_offers_for_id(42)
|> add_metadata()
|> process()
```

It reads like prose, doesn't it?

Every expression in Elixir returns a value. Pipeline operator takes the previous value, so a value that is above it, or on it's left side, and _pipes_ it as a first argument to the following function.

What's great about it is that your functions don't need to be that special. If you write a group of functions that manipulate the same data type, you may follow the simple convention of accepting it as the first argument and returning it as the result. And then your functions compose beautifully like lego blocks.

It's also worth saying that each line is a separate statement, separate function invocation. You are not composing a giant chain of lazily executed computation. You are just executing one statement at the time. Whenever you get an error, you can directly relate a line number to a function call.

That's it for pipelines. But that's not all for Elixir.

Another technique of making working with immutable variables easy is _rebinding_.

In Elixir there is no such thing as a mutable variable, so you are free to do the following:

```elixir
offers = fetch_offers_for_id(42)
offers = add_metadata(offers)
```

This seems to be confusing to newcomers. You may think: _I thought data in Elixir is immutable, what the hell?_

Well, yes - data is in fact immutable. The offers returned by the first function reside in some memory and there is no way your program can change it. In the second line you pass this data as an argument and get some new data in the new memory location[^1].

So the second line doesn't update any memory. It's simply rebinding a name to something new.
We may not be used to it in programming, because you can not do that with _"constant values"_ in mainstream languages. But it's a perfectly normal and convenient thing to do.

In fact, we do it all the time in our lifes.

There is a saying: _"No man can enter the same river twice"_. And it is true. Nothing is persistent. Whenever you enter a river, you are in contact with some particles of water. If you do that again, all of those particles are already long time gone. You are entering a completely different water, a different river. Yet, for our conveniece, we give it the same name - a _Hudson River_.

It is perfectly fine for us, humans, to give a name to a concept that is changing over time.
It would be quite strange to be forced to verbously give a different name to things based on their current state. Imaging saying _me_just_woken_up_, _me_before_coffee_, _me_drinking_coffee_, _me_working_ just because you are transitioning from one state to another. Crazy, isn't it? Yet mainsteam languages force us to do this when dealing with immutable data.

Immutability and name rebinding in Elixir are actually very alike things in our world. You can not change the past (_immutability_) and you are free to give the same names to concepts, even if they change over time (_name rebinding_).

That's all I have to say about immutability in Elixir. I hope you enjoyed it!

[^1]: Virtual machine can take advantage of the fact that both of those pieces of data can not be mutated and if they share any common part - they will both point to the same memory location. Nothing needs to be copied, so memory usage is very effecient.
