# Go 工程师视角：系统学习 gRPC

如果你已经写过：

* REST API（Gin、Echo、Fiber）
* 数据库 CRUD
* 微服务

那么学习 gRPC 最好的方式就是：

> 把它理解成「跨进程的函数调用」。

你调用本地函数：

```go
user, err := GetUser(1001)
```

而 gRPC 本质上是：

```go
user, err := UserService.GetUser(1001)
```

只不过这个函数运行在另一台机器上。

---

# 一、什么是 gRPC

gRPC 是 Google 开源的高性能 RPC 框架。

RPC：

```text
Remote Procedure Call
远程过程调用
```

意思是：

```text
A机器
    ↓
调用
    ↓
B机器上的函数
```

而开发者感觉像在调用本地函数。

---

## 传统 HTTP API

前端请求：

```http
GET /api/user/1001
```

服务端返回：

```json
{
  "id":1001,
  "name":"Tom"
}
```

---

## gRPC

客户端：

```go
client.GetUser(ctx, req)
```

服务端：

```go
func (s *UserService) GetUser(...)
```

感觉像直接调用函数。

---

# 二、gRPC解决了什么问题

大型系统：

```text
用户服务
订单服务
支付服务
库存服务
消息服务
```

服务之间每天互相调用几千万次。

如果都用 REST：

```text
JSON解析
HTTP1.1
文本传输
```

性能损耗较大。

---

gRPC采用：

```text
HTTP/2
+
Protocol Buffers
```

速度更快。

---

# 三、gRPC核心组件

## 1 Protocol Buffers

简称：

```text
protobuf
```

类似：

```text
MySQL -> SQL
gRPC -> protobuf
```

先定义数据结构。

例如：

```proto
message User {
  int64 id = 1;
  string name = 2;
}
```

自动生成：

```go
type User struct {
    Id   int64
    Name string
}
```

---

## 2 Service

定义接口。

```proto
service UserService {
  rpc GetUser(GetUserRequest)
      returns (UserResponse);
}
```

相当于 Go：

```go
type UserService interface {
    GetUser(...)
}
```

---

## 3 Client

客户端。

```go
client.GetUser(...)
```

---

## 4 Server

服务端。

```go
type UserServer struct{}
```

实现接口。

---

# 四、gRPC工作流程

```text
客户端
    │
    ▼
protobuf编码
    │
    ▼
HTTP/2传输
    │
    ▼
服务端
    │
    ▼
protobuf解码
    │
    ▼
执行函数
```

---

# 五、真实业务案例

假设做一个电商系统。

有两个服务：

```text
order-service
user-service
```

订单服务需要查询用户信息。

---

流程：

```text
订单服务
    ↓
调用
    ↓
用户服务
    ↓
返回用户信息
```

---

# 六、创建项目

目录：

```text
grpc-demo/

├── proto/
│   └── user.proto
│
├── server/
│   └── main.go
│
├── client/
│   └── main.go
│
└── gen/
```

---

# 七、编写 proto 文件

user.proto

```proto
syntax = "proto3";

package user;

option go_package = "./gen";

message GetUserRequest {
  int64 id = 1;
}

message User {
  int64 id = 1;
  string name = 2;
  int32 age = 3;
}

service UserService {
  rpc GetUser(GetUserRequest)
      returns (User);
}
```

---

逐行解释：

```proto
syntax = "proto3";
```

使用 protobuf3 语法。

---

```proto
package user;
```

包名。

---

```proto
option go_package="./gen";
```

生成 Go 文件位置。

---

```proto
message User
```

类似：

```go
struct User
```

---

```proto
int64 id = 1;
```

字段编号。

注意：

```text
1
2
3
```

不能随便修改。

因为网络协议依赖编号。

---

# 八、生成代码

安装：

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest

go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

生成：

```bash
protoc \
--go_out=. \
--go-grpc_out=. \
proto/user.proto
```

生成：

```text
user.pb.go
user_grpc.pb.go
```

里面包含：

```go
type User struct{}

type UserServiceClient interface{}

type UserServiceServer interface{}
```

---

# 九、实现服务端

server/main.go

```go
package main

import (
    "context"
    "log"
    "net"

    pb "grpc-demo/gen"

    "google.golang.org/grpc"
)
```

---

实现服务：

```go
type UserServer struct {
    pb.UnimplementedUserServiceServer
}
```

为什么嵌入：

```go
UnimplementedUserServiceServer
```

因为未来 proto 新增方法时不会编译失败。

---

实现接口：

```go
func (s *UserServer) GetUser(
    ctx context.Context,
    req *pb.GetUserRequest,
) (*pb.User, error) {

    return &pb.User{
        Id:   req.Id,
        Name: "Tom",
        Age:  20,
    }, nil
}
```

逐行解释：

---

获取请求参数：

```go
req.Id
```

相当于：

```json
{
  "id":1001
}
```

---

返回：

```go
&pb.User{}
```

会自动序列化成 protobuf。

---

启动服务：

```go
func main() {

    lis, err := net.Listen(
        "tcp",
        ":50051",
    )
    if err != nil {
        log.Fatal(err)
    }

    grpcServer := grpc.NewServer()

    pb.RegisterUserServiceServer(
        grpcServer,
        &UserServer{},
    )

    log.Println("grpc server start")

    grpcServer.Serve(lis)
}
```

---

关键步骤：

监听端口：

```go
net.Listen("tcp", ":50051")
```

---

创建gRPC服务：

```go
grpc.NewServer()
```

---

注册服务：

```go
RegisterUserServiceServer(...)
```

---

启动：

```go
Serve(...)
```

---

# 十、实现客户端

client/main.go

```go
package main

import (
    "context"
    "log"

    pb "grpc-demo/gen"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
)
```

---

连接服务：

```go
conn, err := grpc.Dial(
    "localhost:50051",
    grpc.WithTransportCredentials(
        insecure.NewCredentials(),
    ),
)
```

说明：

```text
开发环境关闭TLS
生产不要这样做
```

---

创建客户端：

```go
client := pb.NewUserServiceClient(conn)
```

---

发送请求：

```go
resp, err := client.GetUser(
    context.Background(),
    &pb.GetUserRequest{
        Id: 1001,
    },
)
```

相当于：

```text
RPC调用
GetUser(1001)
```

---

打印结果：

```go
log.Println(resp)
```

结果：

```text
id:1001
name:"Tom"
age:20
```

---

# 十一、四种RPC模式

这是面试高频。

---

## 1 Unary RPC

最常见。

```text
请求一次
响应一次
```

```text
Client -> Server
```

例如：

```go
GetUser()
```

---

## 2 Server Streaming

```text
请求一次
返回多次
```

例如：

```text
导出100万条订单
```

客户端：

```go
stream.Recv()
```

持续接收。

---

## 3 Client Streaming

```text
发送多次
返回一次
```

例如：

```text
上传大文件
```

客户端不断发送分片。

---

## 4 Bidirectional Streaming

```text
双向流
```

类似：

```text
WebSocket
```

例如：

```text
聊天系统
实时IM
AI对话
```

---

# 十二、生产环境架构

真实公司一般：

```text
API Gateway

      ↓

Order Service
      ↓
      ↓ gRPC
      ↓

User Service

Inventory Service

Payment Service
```

很多互联网公司内部服务调用：

```text
90%以上
都是gRPC
```

而对外仍然：

```text
REST API
```

---

# 十三、生产环境常见坑

## 坑1：Proto字段编号乱改

错误：

```proto
string name = 1;
```

改成：

```proto
string name = 2;
```

可能导致老服务解析错误。

原则：

```text
字段号永不复用
```

删除字段：

```proto
reserved 1;
```

---

## 坑2：超时不设置

错误：

```go
client.GetUser(ctx, req)
```

服务挂了：

```text
无限等待
```

正确：

```go
ctx, cancel := context.WithTimeout(
    context.Background(),
    3*time.Second,
)
defer cancel()
```

---

## 坑3：连接频繁创建

错误：

```go
每次请求都 grpc.Dial()
```

后果：

```text
连接爆炸
CPU飙升
```

正确：

```go
全局连接池
复用连接
```

---

## 坑4：忽略 Context

服务端：

```go
select {
case <-ctx.Done():
    return nil, ctx.Err()
}
```

客户端超时后：

```text
立即停止计算
```

避免资源浪费。

---

## 坑5：消息过大

默认限制：

```text
4MB左右
```

大文件传输：

```text
不要直接RPC
```

正确方案：

```text
文件上传OSS/S3
RPC只传URL
```

---

## 坑6：错误码乱返回

不要：

```go
errors.New("用户不存在")
```

应该：

```go
status.Error(
    codes.NotFound,
    "user not found",
)
```

这样客户端能精确判断。

---

# 十四、企业级 gRPC 技术栈

一个成熟项目通常会配套：

* gRPC
* protobuf
* etcd（服务发现）
* OpenTelemetry（链路追踪）
* Prometheus（监控）
* Jaeger（Tracing）
* Envoy（代理）
* Kubernetes
* TLS/mTLS

形成完整微服务体系。

---

# 学习路线建议

按下面顺序学习效率最高：

```text
第一阶段
├── protobuf语法
├── Unary RPC
└── Go代码生成

第二阶段
├── Context
├── Metadata
├── Interceptor
└── 错误码

第三阶段
├── Server Stream
├── Client Stream
└── Bidirectional Stream

第四阶段
├── 服务发现(etcd)
├── 负载均衡
├── TLS/mTLS
├── OpenTelemetry
└── Kubernetes部署

第五阶段
├── grpc-gateway
├── 熔断限流
├── 链路追踪
└── 企业级微服务架构
```

当你掌握了 **Unary + Stream + Interceptor + Metadata + TLS + 服务发现** 后，基本已经达到中高级 Go 微服务工程师使用 gRPC 的水平。
