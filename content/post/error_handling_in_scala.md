---
date: 2019-01-17
title: Understand Error Handling in Scala
tags: ["FP", "Scala"]
categories: ["FP"]
---

## What's Exception

`Exception` are objects that defined in Java to represent error scenarios. Unhandled exceptions could terminate the program or show default message from http server.

### Excpetion hirarchy

Since Scala is designed to reuse Java standard or thirdparth library without much effort. It still worth to know the exception hirarachy originated in Java. It will help us deal with error scenarios. 

![Exception hierarachy](/static/error-handling/exception_hierarchy.png)

* `Throwable` is the root of exception hirarchy.
* `Exception` subclasses represent errors that the program can recover from. 
* `Error` subclasses represent errors that a program generally shouldn't expect to catch and recover from, like OutOfMemoryError.
* `RuntimeException` is subclass of `Exception`, they represent programming errors that a program shouldn't generally expect to occur, but could recover from, like NullPointerException.
* Subclasses of `Exception`, other than `RuntimeException`, represent errors that a program could generally meet, like network connection error, file read/write error.

### Checked vs unchecked exceptions

*Checked excpetions* need to be declared in the method that throws it and must be handled by the callers, unless it will fail the compiler.

```java
public FileInputStream(File file) throws FileNotFoundException
```

```java
try {
  FileInputStream fin = new FileInputStream(file);
} catch (FileNotFoundException e) {
  e.getMessage()
}
```

In Java we can also keep throwing the exception without handling it.

The `Error`s and `RuntimeException`s (and their subclasses) are called *unchecked exceptions*, means that:

* they can be thrown "at any time"
* methods don't need to declare that they can throw such an exception
* callers don't have to handle them explicitly.

### Exceptions in Scala

All exceptions in Scala are *unchecked* which means the Scala compiler doesn't require you to declare and handle the exception explicitly. 

As a programming language aimed for functional programming, we should never throw exception in Scala code. It breaks the referential transparency. We have following two functions `foo1` and `foo2`.

```scala
def foo1() = if(false) throw new Exception else 2

def foo2() = {
  val a = throw new Exception
  if (false) a  else 2
}

foo1() // res0: Int = 2
foo2() // java.lang.Exception
```

Calling foo1 will terminate normally, calling foo2 will throw an exception.

We should always use a FP way to handle exception in Scala. I have listed a few options below. 

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

`Validated` is the data type provided by [cats](https://github.com/typelevel/cats). It's widely used in scenarios that need accumulating error information, like:

* input form check
* configuration check

With `Validated`, user can see all the error information at once, don't need to wait to see next error until fixed previous one. Use `Invalid(NonEmptyList(error))` when error happens, use `Valid(value)` when calculation ends properly. `NonEmptyList` is a [semigroup](https://typelevel.org/cats/typeclasses/semigroup.html) defined in cats to fulfill data accumulation.

```scala
def divide(m: Int, n: Int): Validated[NonEmptyList[String], Int] =
  if (n == 0) Invalid(NonEmptyList.of("one error"))
  else Valid(m / n)

val result = (divide4(5, 1), divide4(10, 1)).mapN((_, _))

// result is Invalid(NonEmptyList(one error, one error))
```

### Monad Error

`MonadError` is a monad that abstracts over error handling monad like Option / Either / Try, which means we can writing business code without concerning about the specific error handling monad. In this way, our code becomes more generic and compact.

* `pure`: create a value with a successful result wrapped by the error handling monad.
* `raiseError` –  create a value which represents an error.
* `fromEither / fromTry / fromOption / fromValidated` – convert a specific error handling monad into the related MonadError
* `handleError / handleErrorWith / recover / recoverWith` – handle the error scenario and return a successful value.

One example looks like following code:

```scala
trait CustomError[A] {
  def errorFromString(str: String): A
  def errorFromThrowable(e: Throwable): A
}

object CustomizeError {
  implicit val ThrowableCustomError = new CustomError[Throwable] {
    override def errorFromString(str: String): Throwable =
      new Exception(str)
    override def errorFromThrowable(e: Throwable): Throwable =
      e
  }
}

def divide[F[_], E](m: Int, n: Int)(implicit monadError: MonadError[F, E], error: CustomError[E]): F[Int] =
  if (n == 0) monadError.raiseError(error.errorFromString("Divide by 0"))
  else monadError.pure(m / n)

import cats.implicits._
val divideTry = divide[Try, Throwable](10, 0)
```

### Error Handling with IO Monad

`IO` monad is commonly used while dealing with side effects provided by [cats-effect](https://github.com/typelevel/cats-effect). There is an instance of `MonadError[IO, Throwable]` in cats-effect, all the error handling  is done throught it. The convenient function to handle error scenario.

* `raiseError`: constructs an `IO` which wraps up a Throwable.
* `attemp`: sequence the convertion between exception in IO and Either.

```scala
case class BusinessException(msg: String) extends Exception(msg)

val boom = IO.raiseError(new BusinessException("error happens"))

boom.unsafeRunSync()

boom.attempt.map{
  case Left(BusinessException(msg)) => "business error"
  case _ => "other error"
}.unsafeRunSync()

val result = IO.delay(10 / 0).flatMap(n => IO(println(n)))
```

## Summary

Error handling is one of the most important field in programming. FP provides a more graceful way to do it comparing with OO. Review this post and contact me if you got any thoughts.


## Reference
* [Scala best practice: do not throw exception](https://nrinaudo.github.io/scala-best-practices/referential_transparency/avoid_throwing_exceptions.html)
* [Cats data type: Validated](https://typelevel.org/cats/datatypes/validated.html)
* [The exception hierarchy in Java](https://www.javamex.com/tutorials/exceptions/exceptions_hierarchy.shtml)
* [Handling exceptions in Scala](https://pedrorijo.com/blog/scala-exceptions/)
* [Monad Error for the rest of us](https://www.codacy.com/blog/monad-error-for-the-rest-of-us/)
* [Cats Effect: IO](https://typelevel.org/cats-effect/datatypes/io.html)



