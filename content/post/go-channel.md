---
date: 2017-07-29
title: 理解Golang并发编程
tags: ["Go"]
categories: ["DevOps"]
---

## concurrency vs parallelism

并发和并行是彼此相关的两个概念，并不能完全等价。

在程序中，并发强调的是独立执行的程序的组合；并行强调的是同时执行计算任务[1]。  

计算机核心的数量决定了并行计算的能力，大多数人类作为“单核”动物(老顽童小龙女除外)，可以说自己在并发某些任务，如我在听歌写代码，但是不能说这两件事在并行，参考下图：

![concurrency vs parallelism](/static/go-channel/currency vs parallelism.png)

Golang的并发模型源于Communicating Sequential Processes (CSP)，通过提供goroutine和channel来实现并发编程模式。

---

## Goroutine
Goroutine由Go运行时创建和管理，是用于调度CPU资源的“最小单元”，和OS的线程相比更轻量[2]:

* 内存消耗更低只需2kB初始栈空间，而线程初始要1Mb的空间；
* 由golang的运行时环境创建和销毁，更加廉价，不支持手动管理；
* 切换效率更高等。
Goroutine和线程的关系如下图所示：

![goroutine vs thread](/static/go-channel/goroutine vs thread.png)

我们可以轻松地创建成百上千的goroutine，而不会降低程序的执行效率。
  
通过goroutine可以让一个函数和其他的函数并行执行。可以在函数调用前面加上`go`关键字，方便地创建一个goroutine。  

main函数本身也是一个goroutine[3]。  
举例如下：
```
package main

import "fmt"

func main() {
	fmt.Println("begin main goroutine")
    go hello()
    fmt.Println("end main goroutine")
}

func hello() {
	fmt.Println("begin hello goroutine")
}
```
输出：
```
begin main goroutine
end main goroutine
```
上面的例子中，并不会输出`begin hello goroutine`，这是因为，
通过使用goroutine，我们不需要等待函数调用的返回结果，而会接着执行下面的代码。  
可以在`go hello()`后面添加：
```
time.Sleep(1 * time.Second)
```
就可以正常输出`begin hello goroutine`。

---

## channel
Go提供了一种机制能够使goroutine之间进行通信和同步，它就是channel。  
channel是一种类型，关键字`chan`和channel传输内容的类型共同定义了某一channel。  
定义方式为：`var c chan string = make(chan string)`，也可以简写为：`var c = make(chan string)` 或 `c := make(chan string)`  

通过左箭头`<-`操作符操作channel变量:

- `c <- "ping"`向channel发送一个值为“ping”的字符串，  
- `msg := <- c`接收channel中的一个值，并赋給msg。

```
package main

import (
	"fmt"
	"strconv"
	"time"
)

func main() {
	c := make(chan string)
	go ping(c)
	go print(c)
	var input string
	fmt.Scanln(&input)
}

func ping(c chan string) {
	for i := 0; ; i++ {
		c <- strconv.Itoa(i)
	}
}

func print(c chan string) {
	for {
		<-c
		fmt.Println("reveving: " + <-c)
		time.Sleep(1 * time.Second)
	}
}
```
输出：
```
reveving: 1
reveving: 3
reveving: 5
reveving: 7
reveving: 9
    ...
```

按功能，可以将channel分为只发送或只接收channel，通过修改函数签名的channel形参类型来指定channel的“方向”：
  
* 只允许发送: `func ping(c chan<- string)`  
* 只允许接收: `func print(c <-chan string)`  
* 任何对只发送channel的接收操作和只接收channel的发送操作都会产生编译错误。  
* 不指定方向的channel被称作“双向”channel，可以将“双向”channel最为参数，传递给接收单向channel的函数，反之，则不行。

### unbuffered channel
非缓冲channel，也就是缓冲池大小为0的channel或者同步channel，上面的例子都是非缓冲channel，定义方式为：

* `ch := make(chan int)`
* `ch := make(chan int, 0)`

接收非缓冲channel中的数据时，如果channel中没有数据则接收方被阻塞，如果channel中有数据则发送方被阻塞，直到channel中数据被接收。  
使用非缓冲channel，可以通过数据交换来保证两个goroutine的状态同步。

### buffered channel
缓冲channel只能容纳固定量的数据，当缓冲池满之后，发送发被阻塞，直到数据被接收释放缓冲池，定义如下：

* `ch := make(chan int)`

缓冲channel可以用来限制吞吐量，例子如下：
```
package main

import (
	"fmt"
	"time"
)

// Request struct
type Request struct {
}

var sem = make(chan int, 5)     // Create a buffered channel witch capacity of 5

func main() {
	queue := make(chan *Request)
	go start(queue)
	go serve(queue)
	var input string
	fmt.Scanln(&input)
}

func start(queue chan *Request) {
	for {
		queue <- &Request{}
	}
}

func serve(queue chan *Request) {
	for req := range queue {
		sem <- 1       // Put on signal to channel
		go handle(req) // Don't wait for handle to finish.
	}
}

func handle(r *Request) {
	process(r) // May take a long time.
	<-sem      // Done; enable next request to run.
}

func process(r *Request) {
	fmt.Println("process")
	time.Sleep(4 * time.Second)
}
```
每隔4秒钟，输出：
```
process
process
process
process
process
```

---

## select
针对于channel，Golang提供了一个类似`switch`的功能，即`select`，使用如下：

1. `select`选择第一个就绪的channel进行处理
2. 如果有多个就绪的channel，则随机选择一个channel进行处理
3. 如果没有就绪的channel，则等待直到某一channel就绪
4. 如果有`default`，则在3情形中不会等待，而是立即执行default中的代码

```
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	go ping(ch1)
	go pong(ch2)
	go print(ch1, ch2)
	var input string
	fmt.Scanln(&input)
}

func ping(ch chan int) {
	time.Sleep(2 * time.Second)
	ch<-1
}

func pong(ch chan int) {
	time.Sleep(3 * time.Second)
	ch<-2
}

func print(ch1, ch2 chan int) {
	select {
	case msg := <-ch1:
		fmt.Println(msg)
	case msg := <-ch2:
		fmt.Println(msg)
	}
}
```
两秒钟之后，输出：`1`  
在select语句中添加下面代码：
```
default:
	fmt.Println("nothing received.")
```
输出： `nothing received.`

---

## 总结
Golang将线程抽象出来成为轻量级的goroutine，开发者不再需要过多地关注OS层面的逻辑，终于能够从并发编程中解放出来。  
channel作为goroutine通信的媒介，安全高效的实现了goroutine之间的通信和共享内存。  
用Effetive go中的一句话来总结[4]：

> Do not communicate by sharing memory; instead, share memory by communicating.

## Reference
[1] https://blog.golang.org/concurrency-is-not-parallelism  
[2] http://blog.nindalf.com/how-goroutines-work/  
[3] https://www.golang-book.com/books/intro/10  
[4] https://golang.org/doc/effective_go.html  

