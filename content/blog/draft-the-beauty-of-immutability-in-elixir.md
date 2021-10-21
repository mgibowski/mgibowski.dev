Data in Elixir is immutable. It's not immutable by default. It's simply immutable. There's no way around it.
This fact has some interesting consequences.

Benefits of immutability

Immutability saves you from a lot of unnecessary complexity. Your mind doesn't need to maintain a graph of objects with their states. You can focus on fragments of code in isolation. You are in control. When writing a function things are predictible. There is no way that a function you are calling will change the state of a variable you are passing to it. You can rely this won't happen and you can use that variable freely afterwards.

Immutability is particularly useful when dealing with concurrency.

Let's revise some wording. When we are talking about _parallelism_ we mean two or more threads of execution running at the same time. When we say _concurrency_ the meaning is a little bit different. What this means is that there are two or more logical threads of execution that may run in parallel and need to collaborate between each other. In my native language, polish, the literal translation of concurrency would be "konkurencja". If you translate it back to english, the word you get is "competition". I think this word describes concurrency pretty well, as it's mostly about various threads of execution competing for some common resources.

If you ever wrote concurrent code in Java or C++, you know that it's not simple at all. It takes a lot of effort to avoid race conditions. You need to use mechanisms like locks or semaphores to be able to achieve transactionality within your code and you need to be very careful with that - or you may end up with a deadlock. There are only very few, very simple primitives that can be mutated atomically (like AtomicInt).

Now, let's take a look at Elixir. An example from the book "Elixir in Action" by Sasa Juric:

```
```


This function does A,B,C.

You see? No semaphore, no lock, nothing. Just immutable data and it's perfectly thread-safe. ...


You can use immutable data in most mainstream languages, like Java. However it comes with two big ceveats.

First one is that in those languages immutability is optional. Even if you fully commit to immutability, you will probably eventually need to deal with this problem at some point as you use other's people code, for example in libraries.

Second one is that operating uniquely on immutable data in those languages can become cumbersome.

Let's take a look at an example:

```java
const offers = offerService.fetchOffersForId(42);
const offersWithTimestamps = ...
````
You are forced to give a new name to the same variable, just because the concept you are referring to in your code is at the different point in time.

You may get away from that by using a fluent-api style of invocations. However, that is not a perfect solution either. First of all, it requires very specific design of API, which may not be the case for the code you need to invoke. Secondly, it may deteriorate your debugging experience [to check], as the 1:1 relationship between the number of line and method invocation that is so helpful in logging and debugging is no longer there.

Elixir has much better solutions for that.

First one is the pipeline operator. Which actually is a macro.

```
42
|> OffersService.fetch_offers_for_id()
|> OffersService.annotate_with_timestamps()
|> process()
```

It reads like prose, doesn't it?

Every expression in Elixir returns a value. Pipeline operator takes the previous value, so a value that is above it, or on it's left side, and _pipes_ it as a first argument to the following function.

What's great about it is that your functions don't need to be that special. If you write a group of functions that manipulate the same data type, you may follow the simple convention of accepting it as the first argument and returning it as the result. And then your functions compose beautifully like lego blocks.

It's also worth saying that each line is a separate statement, separate function invocation. You are not composing a giant chain of lazily executed computation. You are just executing one statement. Then another. Then another. Etc... Whenever you get an error, you can directly relate a line number to a function invocation.

That's it for pipelines. But that's not all for Elixir.

Another technique of making working with immutable variables easy is rebinding.

As in Elixir there is no such thing as a mutable variable, you are free to do this:

```elixir
offers = OffersService.fetch_offers_for_id(42)
offers = OffersService.annotate_with_timestamps(offers)
```

This seems to be confusing to newcomers. You may think: I thought data in Elixir is immutable, what the hell?

Well, yes - data is in fact immutable. The offers returned by the first function invocation reside in some memory and there is no way your program can change it. In the second line you pass this data as an argument and get some new data in the new memory location. 

_(Virtual machine can take advantage of the fact that both of those pieces of data can not be mutated and if both of those structures share some common part - they will both refer to the same memory location. Therefore memory usage is quite effecient.)_

So the second line doesn't really mean changing anything. It's simply rebinding a name to something new.
We may not be used to it in programming, because you can not do that with constants in mainstream languages. But it's a perfectly normal and convenient thing to do.

In fact, we do it all the time in our lifes.

There is a saying: "No man can enter the same river twice". And it is true. Nothing is persistent. Whenever you enter a river, you are in contact with some particles of water. If you do that again, all of those particles are already long time gone. You are entering a completely different water, a different river. Yet, for our conveniece we give it the same name - a Hudson River.

It's the same with everything. We call one city Tokio and we were calling it with the same name 300 years ago. Although most of the buildings and all of the inhabitants have changed multiple times in between.

It is perfectly fine for us humans to give a name to a concept that is changing over time.
It would be quite strange to be forced to verbously give a different name to things based on their current state. Imaging saying "me_just_woken_up", "me_before_coffee", "me_drinking_coffee", "me_working" just because you are transitioning from one state to another. Crazy, isn't it?

Immutability and name rebinding in Elixir is actually like the world works. You can not change the past (immutability) and you are free to give the same names to concepts, even if they change over time (name rebinding).




Glossary


* variables



/*In the circles of practictioners of "pure" functional programming, probably the most sacred property of the code is referential-transparency. It is said to enable reasoning about code. A requirement for that is separating side effects. In my experience, the most common side effect is data mutation.*/



