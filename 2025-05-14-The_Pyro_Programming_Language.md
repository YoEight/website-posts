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
