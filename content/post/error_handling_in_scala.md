---
date: 2019-01-17
title: Understand Error Handling in Scala
---

## What is error


## Exception model


## How to handle exception

### try catch
We should avoid using `try catch` except calling some unsafe or Java oriented API that provided by thirdparty library. Instead of using `try catch`, it's recommended to use `Try` which will be explained in another section.
```scala
def divide(m: Int, n: Int): Int =
  m / n

val result = try {
  divide(10, 0)
} catch {
  case e: Exception => e.getMessage
}

// result is string: / by zero
```

### Option
When you don't care about the error information, it's ok to use `None` along with `Option` monad to represent the uncared error, and use `Some(value)` to represent the normal calculation result of a function like following example.
```scala
def divide(m: Int, n: Int): Option[Int] =
  if (n == 0) None
  else Some(m / n)

val result = for {
  a <- divide(10, 3)
  b <- divide(7, 0)
} yield a + b

// result is None
```

### Either
`Either` monad is one of the most frequently used error handling methods. Use `Left(error)` to represent the error include type inforamtion, `Right(value)` to represent the normal result.
```scala
def divide(m: Int, n: Int): Either[String, Double] =
  if (n == 0) Left("Divident can't be zero")
  else Right(m / n)

val result = for {
  a <- divide(10, 3)
  b <- divide(7, 0)
} yield a + b

// result is Left("Divident can't be zero")
```

### Try
`Try` monad is another commonly used  way to do error handling in Scala. It's used to dealing with some code which will throw exception out of your control like thirdparty library oriented in Java. Better than `try catch` block, you can get the monad control flow as a bonus. Refer to [this blog](https://whisperd.tech/post/understand-monad/) to get more knowledge about monad control flow. With `Try` monad, you can use `Failure(error)` for error and `Success(value)` for nomal value.
```scala
def divide(m: Int, n: Int): Try[Int] =
  Try{
    m / n
  }

val result = for {
  a <- divide(10, 0)
  b <- divide(10, 3)
} yield a + b

// result is Failure(java.lang.ArithmeticException: / by zero)
```

### Validated



### Monad Error


### Error Handling with IO Monad


## Reference
* [Scala best practice: do not throw exception](https://nrinaudo.github.io/scala-best-practices/referential_transparency/avoid_throwing_exceptions.html)



