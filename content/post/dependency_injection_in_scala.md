---
date: 2019-01-17
title: Dependency Injection in Scala
tags: ["FP", "Scala"]
categories: ["FP"]
---

## What's Dependency Injection

Dependency Injection is also known as DI for short. It is all about the way to integrate code between provider and consumer. 

Usually, the provider provides functionalities that encapsulated in a function or an object. The consumer, in the opposite it needs to have a provider to do some work. 

Then we say that the consumer dependent on the specific provider. There are two options that we can get this provider:

* construct the provider in consumer.
* inject the provider to consumer when calling the consumer.

Injection is better since it help us to do split concern (and conform with SRP). As a result, it makes our code easier to understand, more testable and more reusable.

Different programming language have different way or library to do DI, but the idea behind it is nearly same.

## How to do DI in Scala

Functional programming is all about make function pure, and pure function has two rules:

* Output only depend on input
* No side effect

That means we prefer to use DI and should never use global state. I have listed a few common ways to do DI in Scala.

### Cake pattern

There is a saying that the cake pattern is dead. You can skip this section without hesitation. If you are still interested in this feature, keep reading from here.

First let's introduce *Self-type* identifier `self: AnotherTrait =>` at first line of class or trait definition. 

```scala
trait A {
  def doAStuff = "in A"
}

trait B {
  self: A =>
  def doBStuff = s"in B, ${doAStuff}"
}
```

It means that you can mixed in the trait B only if you mixed in trait A first. `self` is just an alias for `this` keyword, using `this` will cause [issue](https://stackoverflow.com/questions/4017357/difference-between-this-and-self-in-self-type-annotations/4018995#4018995) when access the outer scope from inner class.

```scala
case class AppConfig(host: String)

trait Config {
  def get: AppConfig
}

class Service {
  self: Config =>
  def run = println(s"config: $get")
}

val service = new Service with Config {
  def get: AppConfig = AppConfig("http://example.com")
}

service.run  // config: AppConfig(http://example.com)
```

In above code, we need to give implementation for each injected trait when instantiate the `Service` class or trait. Here is a more practical way to do it.

```scala
trait ConfigComponent {
  val config: Config
  
  class Config {
    def get: AppConfig = AppConfig("http://example.com")
  }
}

class Service {
  self: ConfigComponent =>
  def run = println(s"config: ${config.get}")
}

val service = new Service with ConfigComponent {
  val config = new Config
}

service.run  // config: AppConfig(http://example.com)
```

If there is multiple dependencies, you can simply use `with` between the traits.

### Reader monad



### Macwire

### Guice



### Tagless final



### Eff monad



## References

* [macwire](https://github.com/adamw/macwire)
* [DI in Scala guide](http://di-in-scala.github.io/)
* [The cake pattern is a lie](https://mikulskibartosz.name/the-cake-pattern-is-a-lie-2fe1cfd7dea9)
* [Scala: Self-type](https://docs.scala-lang.org/tour/self-types.html)
* [Disambiguating 'this' in Scala](http://enear.github.io/2018/10/08/self-arrow/)