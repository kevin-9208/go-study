go语言介绍

如果把 Go（Golang）比作一门武功，那么很多人学习 Go 的路线其实是：

> 学语法 → 写 CRUD → 学协程 → 学 Web → 学数据库 → 学微服务 → 学云原生 → 学源码

结果经常出现：

* 会写代码，但不理解 Go 的设计哲学
* 会用 goroutine，但不知道调度器原理
* 会用 channel，但不知道什么时候不用 channel
* 会用 Gin，但不会写标准库 net/http
* 会用 gRPC，但不理解 Go 在云原生中的定位

所以我建议按照：

**语言基础 → 核心机制 → 工程实践 → 并发编程 → 网络编程 → 微服务 → 云原生**

这样的顺序学习。

---

# 第一阶段：认识 Go

## Go 是什么？

Go 是由 Robert Griesemer、Rob Pike、Ken Thompson 在 [Google](https://www.google.com?utm_source=chatgpt.com) 设计的一门语言。

目标：

* 像 C 一样快
* 像 Python 一样简单
* 天然支持并发
* 适合大型工程

特点：

```text
简单
高性能
编译型
静态类型
垃圾回收
并发友好
跨平台
```

---

# 第二阶段：开发环境

安装 Go

官方：

[Go 官方网站](https://go.dev?utm_source=chatgpt.com)

查看版本：

```bash
go version
```

查看环境：

```bash
go env
```

创建项目：

```bash
mkdir hello-go
cd hello-go

go mod init hello-go
```

目录：

```text
hello-go
 ├─ go.mod
 └─ main.go
```

---

# 第一个程序

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello Go")
}
```

运行：

```bash
go run .
```

编译：

```bash
go build
```

生成：

```text
hello-go.exe
```

---

# 第三阶段：基础语法

---

## 变量

### 普通声明

```go
var name string = "Tom"
```

### 类型推导

```go
var name = "Tom"
```

### 短变量

最常用：

```go
name := "Tom"
```

相当于：

```go
var name string = "Tom"
```

---

## 基础类型

```go
int
int8
int16
int32
int64

uint
uint8

float32
float64

bool

string
```

示例：

```go
var age int = 18
var price float64 = 19.9
var ok bool = true
```

---

## 常量

```go
const PI = 3.14
```

---

## iota

Go 特有枚举写法：

```go
const (
	Monday = iota
	Tuesday
	Wednesday
)
```

结果：

```text
0
1
2
```

---

# 第四阶段：流程控制

## if

```go
if age > 18 {
	fmt.Println("adult")
}
```

---

## switch

Go 的 switch 很强大。

```go
switch score {
case 90:
	fmt.Println("A")
case 80:
	fmt.Println("B")
default:
	fmt.Println("C")
}
```

无需 break。

---

## for

Go 没有 while。

### 普通循环

```go
for i := 0; i < 10; i++ {
	fmt.Println(i)
}
```

---

### while 写法

```go
for n < 10 {
	n++
}
```

---

### 无限循环

```go
for {
}
```

---

# 第五阶段：函数

## 普通函数

```go
func add(a int, b int) int {
	return a + b
}
```

---

## 多返回值

Go 特色之一。

```go
func div(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("divide by zero")
	}

	return a / b, nil
}
```

使用：

```go
res, err := div(10, 0)
```

---

## 命名返回值

```go
func sum(a, b int) (result int) {
	result = a + b
	return
}
```

---

## 可变参数

```go
func sum(nums ...int) int {
}
```

---

# 第六阶段：指针

Go 有指针。

但没有 C 那么危险。

```go
a := 10

p := &a
```

取值：

```go
fmt.Println(*p)
```

修改：

```go
*p = 100
```

结果：

```go
a == 100
```

---

# 第七阶段：数组与切片

## 数组

长度固定：

```go
var nums [3]int
```

```go
[1,2,3]
```

---

## Slice（切片）

Go 最重要的数据结构之一。

```go
nums := []int{1,2,3}
```

追加：

```go
nums = append(nums, 4)
```

切片：

```go
nums[1:3]
```

---

## len 和 cap

```go
len(slice)
cap(slice)
```

示例：

```go
s := make([]int, 3, 10)

len=3
cap=10
```

---

# 第八阶段：Map

类似：

```text
Java HashMap
Python dict
JS Object
```

创建：

```go
m := make(map[string]int)
```

赋值：

```go
m["Tom"] = 18
```

读取：

```go
age := m["Tom"]
```

判断存在：

```go
age, ok := m["Tom"]
```

---

# 第九阶段：Struct

Go 没有 class。

最接近的是 struct。

```go
type User struct {
	Name string
	Age  int
}
```

创建：

```go
u := User{
	Name: "Tom",
	Age: 18,
}
```

---

# 第十阶段：方法 Method

给结构体绑定方法。

```go
func (u User) SayHello() {
	fmt.Println("hello")
}
```

调用：

```go
u.SayHello()
```

---

# 第十一阶段：接口 Interface

这是 Go 最核心的设计之一。

定义：

```go
type Speaker interface {
	Speak()
}
```

实现：

```go
type Dog struct{}

func (Dog) Speak() {
	fmt.Println("wang")
}
```

无需显式实现：

```go
var s Speaker = Dog{}
```

这叫：

```text
Duck Typing
鸭子类型
```

---

# 第十二阶段：错误处理

Go 不使用异常作为主要错误机制。

经典模式：

```go
result, err := doSomething()

if err != nil {
	return err
}
```

定义错误：

```go
errors.New("something wrong")
```

包装错误：

```go
fmt.Errorf("query user: %w", err)
```

判断：

```go
errors.Is()
errors.As()
```

---

# 第十三阶段：包管理

## 创建模块

```bash
go mod init demo
```

---

## 安装依赖

```bash
go get github.com/gin-gonic/gin
```

---

## 更新依赖

```bash
go mod tidy
```

---

# 第十四阶段：泛型（Go 1.18+）

以前：

```go
func SumInt([]int)
func SumFloat([]float64)
```

泛型：

```go
func Sum[T int | float64](nums []T) T {
}
```

使用：

```go
Sum([]int{1,2,3})
```

---

# 第十五阶段：并发编程

这是 Go 的王牌。

---

## goroutine

启动协程：

```go
go func() {
	fmt.Println("hello")
}()
```

仅需一个 go 关键字。

---

## channel

线程安全通信。

创建：

```go
ch := make(chan int)
```

发送：

```go
ch <- 1
```

接收：

```go
v := <-ch
```

---

## select

多路复用：

```go
select {
case msg := <-ch1:
	fmt.Println(msg)

case msg := <-ch2:
	fmt.Println(msg)
}
```

---

## WaitGroup

等待任务结束：

```go
var wg sync.WaitGroup
```

---

## Context

现代 Go 必学。

```go
ctx, cancel := context.WithTimeout(
	context.Background(),
	5*time.Second,
)
```

用于：

* 超时
* 取消
* 请求链路传递

---

# 第十六阶段：反射

运行时获取类型信息。

```go
reflect.TypeOf(v)
reflect.ValueOf(v)
```

应用：

* ORM
* JSON库
* 框架

例如：

```go
json.Marshal()
```

内部大量使用反射。

---

# 第十七阶段：标准库

Go 强大的地方之一：

很多场景不用第三方库。

---

## JSON

```go
encoding/json
```

```go
json.Marshal()
json.Unmarshal()
```

---

## HTTP

```go
net/http
```

服务端：

```go
http.ListenAndServe(":8080", nil)
```

客户端：

```go
http.Get(url)
```

---

## 文件

```go
os
io
bufio
```

---

## 日志

```go
log
slog
```

Go 1.21 推荐：

```go
log/slog
```

---

# 第十八阶段：Go Web 开发

主流框架：

* [Gin](https://gin-gonic.com?utm_source=chatgpt.com)
* [Echo](https://echo.labstack.com?utm_source=chatgpt.com)
* [Fiber](https://gofiber.io?utm_source=chatgpt.com)

Gin 示例：

```go
r := gin.Default()

r.GET("/hello", func(c *gin.Context) {
	c.JSON(200, gin.H{
		"msg":"hello",
	})
})

r.Run()
```

---

# 第十九阶段：数据库

## MySQL

驱动：

```go
github.com/go-sql-driver/mysql
```

标准库：

```go
database/sql
```

---

## ORM

主流：

[GORM](https://gorm.io?utm_source=chatgpt.com)

```go
db.Find(&users)
```

---

# 第二十阶段：gRPC

核心组件：

```text
Proto
Client
Server
HTTP/2
CodeGen
```

典型流程：

```text
proto
 ↓
protoc
 ↓
Go代码
 ↓
Server
 ↓
Client
```

这是现代微服务的主流通信方式之一。

---

# 第二十一阶段：Go 云原生生态

Go 几乎统治了云原生基础设施领域。

很多知名项目都是 Go 编写：

* Docker
* Kubernetes
* etcd
* Prometheus
* Terraform
* Consul

---

# 第二十二阶段：进阶路线

当你掌握上面内容后，建议继续深入：

### Go Runtime

重点：

```text
GMP模型
调度器
栈扩容
GC
内存管理
```

---

### 网络源码

```text
net
net/http
http2
grpc
```

---

### 高并发

```text
sync.Mutex
RWMutex
sync.Map
atomic
pool
```

---

### 源码阅读

推荐顺序：

```text
context
sync.WaitGroup
channel
map
net/http
grpc
gin
gorm
```

---

# 一份比较合理的学习路线（3~6个月）

```text
第一周
├── 基础语法
├── 函数
├── struct
└── interface

第二周
├── slice
├── map
├── error
└── package

第三周
├── goroutine
├── channel
├── context
└── sync

第四周
├── json
├── 文件操作
├── net/http
└── 项目实战

第二个月
├── MySQL
├── Redis
├── Gin
├── JWT
└── REST API

第三个月
├── gRPC
├── Docker
├── Kubernetes
└── 微服务

第四个月+
├── Runtime
├── GC
├── 调度器
├── 源码阅读
└── 云原生
```

如果你希望真正达到**高级 Go 工程师**水平，下一步最值得学习的不是更多语法，而是 Go 的 **内存模型、Slice 底层原理、Map 实现、Interface 原理、GMP 调度器、Channel 实现、GC 机制、Context 设计思想**。这些内容决定了你能否读懂大型项目（如 Kubernetes、Docker、etcd、Prometheus）的源码。
