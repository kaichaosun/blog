---
date: 2019-01-17
title: Dependency Injection in Scala
---

## What's Dependency Injection

Dependency Injection is also known as DI for short. It is all about the way to integrate code between provider and consumer. 

Usually, the provider  provides functionalities that encapsulated in a function or an object. The consumer in another direction that needs to have a provider whenever necessary. 

Then we say that the consumer dependent on the specific provider. There are two options that we can get this provider:

* construct the provider in consumer.
* inject the provider to consumer when calling the consumer.

Injection is better since it help us to do split concern. As a result, it makes our code easier to understand, more testable and more reusable.

Different programming language have different way or library to do DI, but the idea behind it is nearly same.

## How to do DI in Scala



### Cake pattern



### Reader monad



### Macwire



### Tagless final



### Eff monad



## References

* [macwire](https://github.com/adamw/macwire)
* [DI in Scala guide](http://di-in-scala.github.io/)