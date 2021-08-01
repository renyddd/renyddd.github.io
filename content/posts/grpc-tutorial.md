---
title: "Grpc Tutorial"
date: 2021-07-14T22:37:45+08:00
draft: false
tags: ["golang", "grpc"]
categories: ["golang"]
series: ["golang系列"]
---

# gRPC Tutorial

首先参考[官方 Basics Tutorial](https://grpc.io/docs/languages/go/basics/) 来完成一次简单的构建，本文以下内容皆是对官方文档的模仿：

1. 定义自己的 service `.proto` 文件；
2. 通过 protocol buffer compiler 生成 C/S 端代码；
3. 为你的 service 通过 go gRPC API 编写你的端逻辑。

**TODO** gRPC 简介：https://grpc.io/docs/what-is-grpc/introduction/

## 代码部分

本次所构建的为一个服务端响应客户端 “问好 greeting” 的例子，首先从定义一个 service 开始：

```protobuf
// tutorial.proto
service Greeter {
  rpc SayHello1 (HelloRequest) returns (HelloReply) {}
}
```

接着你可以为该 service 定义方法（method），并且同时需要指明 request 和 response 类型。

### service

gRPC 给你提供了 4 种 service method，分别是：

1. A simple RPC：客户端通过 stub（活着称为 client）来向服务端发送请求，并且等待着回应，就像普通的函数调用一样；

   ```protobuf
   rpc SayHello1 (HelloRequest) returns (HelloReply) {}
   ```

2. A server-side streaming RPC：客户端发送请求并获得一个 stream 来读取一系列的信息，服务端从该 stream 中等到再无信息后才停止，如下所示将关键字 **stream** 放至 response type 前即可；

   ```protobuf
   rpc Method2 (Request2) return (stream Reply2) {}
   ```

3. A server-side streaming RPC：客户端写入一系列的信息并且发送给服务端，同样是通过提供的 stream；一旦客户端完成了信息的写入，便会等待服务端的读取完毕与响应，通过将 **stream** 关键字放置在 request type 前即可；

   ```protobuf
   rpc Method3 (stream Request3) return (Reply3) {}
   ```

4. A *bidirectional streaming RPC* where both sides send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved. You specify this type of method by placing the `stream` keyword before both the request and the response.



### message

你的 `.proto` 文件中同样包含有 protocol buffer 的 message 类型的定义，以为你的 service method 中的 request、response 类型中所使用，例如：

```protobuf
message HelloRequest {
  string name = 1;
}

message HelloReply {
  string msg = 1;
}
```

### 代码生成

接下来我们通过 protoc 工具，来从上述文件的 service 定义生成客户端和服务端接口。假设我们当前所在的目录为 `protocolbuffers`，执行如下命令：

```bash
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative ./tutorial.proto
```

过后，会在当前目录下生成如下两个文件：

- tutorial.pb.go：包含有填充、序列化和收发响应请求的全部 protocol buffer 代码；
- tutorial_grpc.pb.go：包含有如下两部分：
  - 一个客户端的接口类型 GreeterClient，与用来调用定义在 Greeter service 中的 method；
  - 一个待实现的服务端接口类型，与定义在 Greeter service 中的 method。

```go
// tutorial_grpc.pb.go
type GreeterClient interface {
	SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error)
}

type greeterClient struct {
	cc grpc.ClientConnInterface
}

func (c *greeterClient) SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error) {
	out := new(HelloReply)
	err := c.cc.Invoke(ctx, "/protocolbuffers.Greeter/SayHello", in, out, opts...)
	if err != nil {
		return nil, err
	}
	return out, nil
}

/*
...
*/

// GreeterServer is the server API for Greeter service.
// All implementations must embed UnimplementedGreeterServer
// for forward compatibility
type GreeterServer interface {
	SayHello(context.Context, *HelloRequest) (*HelloReply, error)
	mustEmbedUnimplementedGreeterServer()
}

// UnimplementedGreeterServer must be embedded to have forward compatible implementations.
type UnimplementedGreeterServer struct {
}

func (UnimplementedGreeterServer) SayHello(context.Context, *HelloRequest) (*HelloReply, error) {
	return nil, status.Errorf(codes.Unimplemented, "method SayHello not implemented")
}
func (UnimplementedGreeterServer) mustEmbedUnimplementedGreeterServer() {}
```

### 添加逻辑

你可以看到代码生成的服务端结构体为 Unimplement 未实现（状态），UnimplementedGreeterServer 结构体虽然实现了 GreeterServer 接口，但其 SayHello 方法却并无实际服务端逻辑。

故我们在此创建服务端的操作应为：

1. 嵌入 UnimplementedGreeterServer 以实现 GreeterServer 接口；
2. 添加实际业务逻辑。

```golang
import gw "github.com/renyddd/golang/pbApp/server/v1"
import "google.golang.org/grpc"

type Server struct {
	gw.UnimplementedGreeterServer
  // 自定义问好前缀
	greetingPrefix string
}

func (s *Server) SayHello(ctx context.Context, in *pb.HelloRequest) (*gw.HelloReply, error) {
	log.Println("[log] server ", s, in)
	return &gw.HelloReply{
		Msg: s.greetingPrefix + in.Name,
	}, nil
}
```

### 启动服务端

指定监听端口，创建 grpc.Server 实例，并向该实例注册我们的服务，启动服务阻塞至该进程被杀：

```golang
func main() {
  ServerWithoutGateway()
}

func ServerWithoutGateway() {
	lis, err := net.Listen("tcp", "localhost:8081")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	var opts []grpc.ServerOption
	grpcServer := grpc.NewServer(opts...)
	gw.RegisterGreeterServer(grpcServer, newServer())
	grpcServer.Serve(lis)
}
```

### 启动客户端

```golang
import pb "github.com/renyddd/golang/somePkg/pbApp/server/v1"
import "google.golang.org/grpc"

func Rpc_SayHello(ctx context.Context, client pb.GreeterClient, msg string) (string, error) {
	hRequest := &pb.HelloRequest{
		Name: msg,
	}
	hReplay, err := client.SayHello(ctx, hRequest)
	if err != nil {
		return "", err
	}

	return hReplay.Msg, nil
}

func main() {
	var opts []grpc.DialOption
	opts = append(opts, grpc.WithInsecure())

	serverAddr := "localhost:8081"
	conn, err := grpc.Dial(serverAddr, opts...)
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	client := pb.NewGreeterClient(conn)

	msg, err := Rpc_SayHello(context.Background(), client, "Alice")
	if err != nil {
		panic(err)
	}
	log.Println("rpc replay: ", msg)
}
```

## TODO

1. [gRPC-gateway](https://github.com/grpc-ecosystem/grpc-gateway)
2. 原理解析





