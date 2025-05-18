# Pyro: A Toy Language Rooted in Temporal Query Modeling

Pyro began as an attempt to find a better way to describe user-defined projections in KurrentDB (formerly EventStoreDB)—specifically, the kind of temporal correlation queries that are difficult to express clearly using existing tools. That exploration led me to design a small programming language inspired by the π-calculus and built with concurrency and messaging as first-class concepts.

While Pyro isn’t meant for production use, it evolved into a valuable playground—one where I could model event streams, experiment with concurrency, and reason about time through code.

## What is  π-calculus ?

The π-calculus is a mathematical model for describing concurrent systems, where multiple processes run independently and communicate with each other. It focuses on how processes exchange messages through channels, and uniquely, it allows those channels to be created and passed around dynamically.

In essence, π-calculus helps us reason formally about systems that evolve through interaction—like distributed programs, network protocols, or event-driven architectures.

In simpler terms, it’s a way to formally reason about how independent parts of a system can send and receive messages, change connections over time, and evolve based on interaction—making it ideal for modeling things like distributed systems, messaging protocols, or event streams.

## Pyro key features

Pyro is a functionally oriented language that avoids mutable state. All data is currently immutable by design. It features a strong nominal type system, where types are explicitly named and checked for compatibility, helping catch many errors at compile time. Type inference is supported for non-top-level declarations, reducing boilerplate without sacrificing clarity. For flexibility during experimentation or rapid prototyping, type checking can be disabled, in which case type errors or references to undefined variables are deferred to runtime and reported as exceptions.

## Core concepts

At the core of Pyro are processes, the basic units of computation that can run concurrently. Processes interact by sending and receiving messages over channels. A sender transmits a message along a channel, while a receiver waits for a message to arrive on a given channel before continuing execution. Channels themselves can be passed as messages, allowing the communication structure of the system to evolve dynamically. Pyro also includes constructs for parallel composition, which allows multiple processes to run simultaneously, and name restriction, which creates new, private channels. Together, these concepts provide a powerful and minimal foundation for modeling complex, concurrent behaviors.

Sending and receiving messages are first-class concepts in the language, each with dedicated syntax to make communication between processes explicit and expressive.

## How does Pyro look like?

```
print ! "Hello, World!"
```

Unfamiliar syntax? Think of `print` as a process, and the `!` operator as a way to send it a message—in this case, the string `"Hello, World!"`. If you’ve used Scala with the Akka framework, this should feel familiar. It’s similar to sending a message to an Actor, where `print` plays the role of the actor in this example.

The following example shows how receiving a message looks like.

```
echo ? msg = print ! msg
```

`echo` is a process, and the `?` operator means “wait for” a message—here, bound to `msg` variable. When a message arrives, the expression after the `=` is executed. In this case, it sends the received `msg` to the `print` process. Pyro supports pattern matching, so one could have written that alternative instead:

```
echo ? [x] = print ! x
```

We expect an array containing a single element and bind that element to the x variable. Finally, we send x to the print process.

Process communicatioin aside, Pyro is very similar to Lisp and use prefix notation for functions:

```
(+ 1 2)
```

Which is the equivalent of the infix notation `1 + 2`.

You can define functions in Pyro, though they’re more like syntactic sugar for creating processes—essentially an abstraction layer. The syntax might look unfamiliar at first,
but it should still be understandable with a bit of context.

```
(def foobar [x y] = (x ! y | x ! y))
```

As a process abstraction, `foobar` can only return another process. For example, you can't return a number. In this case, `foobar` defines a process that waits for a message: specifically, an array containing exactly two elements.
This is how Pyro supports parameters. At its core, a process can only receive a single message, so if you need to pass multiple arguments, you wrap them in an array. The `|` operator
in Pyro represents parallel execution. In this snippet, we send the message `y` to the process `x` twice in parallel.

Let’s look at a more involved example. Pyro is functional by design and doesn’t include traditional looping constructs like `for` or `while`. Instead, looping behavior can be achieved by defining a process abstraction that uses recursion.

```
(def for [min: Integer max: Integer f: ![Integer ^[]] done: ^[]] =
  (def loop x : Integer =
    if (<= x max) then
      (new c : ^[]
        ( f ! [x c] | c?[] = loop ! (+ x 1)))
    else
      done ! []

  loop ! min ))
```

Admittedly, this looks a lot more intimidating than your typical for loop. Let’s break down the syntax to make it more approachable.

`min: Integer` is a standard variable declaration. Arrays in Pyro can have labels, which are optional. Labels introduce variable names bound to their corresponding values, and optionally, their types. When labeling types, `!` denotes a `Client`—a process that can only be sent messages. The `^` symbol represents a `Channel`, which means it can both send and receive messages. In Pyro, a `Client` is write-only, a `Receiver` is read-only, and a `Channel` combines both capabilities.

Now, about `f: ![Integer ^[]]`: this means `f` is a `Client` that expects to receive an array of two elements. The first is an `Integer`, and the second is a `Channel` that handles an empty array. In other words, `f` can be sent a message consisting of a number and a communication endpoint that expects no input.
