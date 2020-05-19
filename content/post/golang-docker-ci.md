---
date: 2017-07-05
title: Golang web 开发
tags: ["Go"]
categories: ["DevOps"]
---

## 背景
Web应用长期以来是Ruby、Java、PHP等开发语言的战场。

- Ruby可以实现快速原型开发，Ruby On Rails “全能”框架实现“全栈”开发，缺点有大型应用性能差、调试困难；
- Java 20多年的发展历程，各种第三方库、框架健全，运行效率高，
但是随着应用的功能膨胀，臃肿的get/set方法，JVM占用大量计算机资源、性能调试困难，函数式编程不友好。
- PHP，TL;DR

本文实现了一个最小化web应用，以此来了解Golang web的生态，通过使用Docker隔离开发环境，
使用Posgres持久化数据，源代码请参考[这里](https://github.com/kaichaosun/golang-web-starter)

---
## Why Go?

- 性能优越
- 部署简单，只需要将打包好的二进制文件部署到服务器上
- 内置丰富的标准库，让程序员的生活变得简单美好
- 静态语言，类型检查
- duck typing
- goroutine将开发人员从并发编程中解放出来
- 函数作为“一等公民”
- ...


## Golang第三方框架选择

* Web框架: [Gin](https://github.com/gin-gonic/gin)，性能卓越，API友好，功能完善
* ORM: [GORM](https://github.com/jinzhu/gorm),支持多种主流数据库方言，文档清晰
* 包管理工具: [Glide](https://github.com/Masterminds/glide),类似于Ruby的bundler或者NodeJS中的npm
* 测试工具:

> * [GoConvey](https://github.com/smartystreets/goconvey),符合BDD测试风格,支持浏览器测试结果的可视化
> * [Testify](https://github.com/stretchr/testify),提供丰富的断言和Mock功能

* 数据库migration: [migrate](github.com/mattes/migrate)
* 日志工具: [Logrus](https://github.com/Sirupsen/logrus),结构化日志输出，完全兼容标准库的logger

## Dockerize 开发环境

### 发布应用 base image
Dockerfile如下:
```
FROM golang:1.8

# 包管理工具
RUN curl https://glide.sh/get | sh  

# 代码热加载    
RUN go get github.com/codegangsta/gin  

# 数据库migration工具
RUN go get -u -d github.com/mattes/migrate/cli github.com/lib/pq
RUN go build -tags 'postgres' -o /usr/local/bin/migrate github.com/mattes/migrate/cli
```

### 发布数据库 base image
Dockerfile如下：
```
FROM postgres:9.6

# 初始化数据库配置
COPY ./init-user-db.sh /docker-entrypoint-initdb.d/init-user-db.sh
```

### 启动服务
运行`auto/dev`即可启动，具体的配置如下。
* docker-compose.yml：

```bash
version: "3"

services:
  dev:
    links:
      - db
    image: 415148673/golang-web-base-image@sha256:18de5eb058a54b64f32d58b57a1eb3009b9ed49d90bd53056b95c5c8d5894cd6
    environment:
      - PORT=8080
      - DB_USER=docker
      - DB_HOST=db
      - DB_NAME=webstarter
    volumes:
      - .:/go/src/golang-web-starter
    working_dir: /go/src/golang-web-starter
    ports:
      - "3000:3000"
    command: gin

  db:
    image: 415148673/postgres@sha256:6d4800c53e68576e05d3a61f2b62ed573f40692bcc72a3ebef3b04b3986bb70c
    volumes:
      - go-web-starter-db-cache:/var/lib/postgresql/data

volumes:
  go-web-starter-db-cache:
```

* 安装第三方依赖所需的glide配置文件，通过在容器内运行`glide install`进行安装：

```
package: golang-web-starter
import:
- package: github.com/gin-gonic/gin
  version: ^1.1.4
- package: github.com/jinzhu/gorm
  version: ^1.0.0
- package: github.com/mattes/migrate
  version: ^3.0.1
- package: github.com/lib/pq
- package: github.com/stretchr/testify
  version: ^1.1.4
- package: github.com/smartystreets/goconvey
  version: ^1.6.2
```

* 数据库migration的脚本：

```
migrate -source file://migrations -database "postgres://$DB_USER:$DB_PASSWORD@$DB_HOST:5432/$DB_NAME?sslmode=disable" up
```

---
## 业务实现
### Router
```go
router := gin.Default()
router.GET("/", handler.ShowIndexPage)        // 显示主页面
router.GET("/book/:book_id", handler.GetBook) // 通过id查询书籍
router.POST("/book", handler.SaveBook)        // 保存书籍
```
### handler
以保存书籍为例：
```go
func SaveBook(c *gin.Context)  {
	var book models.Book
	if err := c.Bind(&book); err == nil {
    // 调用model的保存方法
		book := models.SaveBook(book)   

    // 绑定前端页面所需数据
		utility.Render(
			c,
			gin.H{
				"title": "Save",
				"payload": book,
			},
			"success.html",
		)
	} else {
    // 异常处理
		c.AbortWithError(http.StatusBadRequest, err)
	}
}
```
### model
```go
func SaveBook(book Book) Book {
  // 持久化数据
	utility.DB().Create(&book)
	return book;
}
```
### 建立DB连接
```go
func DB() *gorm.DB {
	dbInfo := fmt.Sprintf(
		"host=%s user=%s dbname=%s sslmode=disable password=%s",
		os.Getenv("DB_HOST"),
		os.Getenv("DB_USER"),
		os.Getenv("DB_NAME"),
		os.Getenv("DB_PASSWORD"),
	)
	db, err := gorm.Open("postgres", dbInfo)
	if err != nil {
		log.Fatal(err)
	}
	return db
}
```
### View
```html
<body class="container">
		{{ template "menu.html" . }}
		<label>保存成功</label>
		<h1>{{.payload.Title}}</h1>
		<p>{{.payload.Author}}</p>
		{{ template "footer.html" .}}
</body>
```
### 测试
```go
func TestSaveBook(t *testing.T) {
	r := utility.GetRouter(true)
	r.POST("/book", SaveBook)

	Convey("The params can not convert to model book", t, func() {
		req, _ := http.NewRequest("POST", "/book", nil)
		req.Header.Set("Content-Type", "application/x-www-form-urlencoded")

		utility.TestHTTPResponse(r, req, func(w *httptest.ResponseRecorder) {
			So(w.Code, ShouldEqual, http.StatusBadRequest)
		})
	})

	Convey("The params can convert to model book", t, func() {
		req, _ := http.NewRequest("POST", "/book", strings.NewReader("title=Hello world&author=will"))
		req.Header.Set("Content-Type", "application/x-www-form-urlencoded")
		utility.TestHTTPResponse(r, req, func(w *httptest.ResponseRecorder) {
			p, _ := ioutil.ReadAll(w.Body)
			So(w.Code, ShouldEqual, http.StatusOK)
			So(string(p), ShouldContainSubstring, "保存成功")
		})
	})
}
```

## 总结
Go生态之活跃令我大开眼界,著名的应用如ocker, Ethereum都是使用Go编写的。

使用Go进行web开发的过程，感觉和搭积木一样，一个合适的第三方库需要在多个候选库中精心筛选, 
众多的开源作者共同构建了一个“模块”王国。在这样的环境中, 编程变成了一件很自由的事情。  

由于Go的标准库提供了很多内置的实用命令如`go fmt`,`go test`,让编程变得异常轻松,简直是强迫型程序员的“天堂”。  

当然Go语言还处在发展过程中,也有许多不完善的地方,比如  

- 缺少标准的依赖管理工具（正在开发的`dep`）
- 非中心化的依赖仓库会出现由于某个依赖被删除导致应用不可用等。
