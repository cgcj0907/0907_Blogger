---
title: "Go gRPC 开发实战笔记"
date: 2025-1-5
draft: false
tags: ["Languages", "Golang", "gRPC"]
ShowToc: true
---

## 1. 核心流程概览
1.  **定义协议 (.proto)**: 描述服务接口和消息结构
2.  **生成代码 (protoc)**: 自动生成 Go 语言的接口代码
3.  **服务端实现 (Server)**: 实现生成的接口逻辑，并启动 gRPC Server
4.  **客户端调用 (Client)**: 建立连接并调用远程方法

## 2. 伪代码框架

你可以直接复制这个框架作为新服务的模板。

### 第一步：定义 Proto 文件
路径: `protobuf/service_name.proto`

```protobuf
syntax = "proto3";

// 指定生成的 Go 代码包路径
option go_package = "path/to/genproto/service_name";

// 1. 定义服务接口
service MyService {
  // 定义方法：接收 Request，返回 Response
  rpc MyMethod(MyRequest) returns (MyResponse) {}
}

// 2. 定义请求消息
message MyRequest {
  string some_id = 1;
  int32  amount  = 2;
}

// 3. 定义响应消息
message MyResponse {
  bool   success = 1;
  string message = 2;
  Data   data    = 3; // 嵌套消息
}

message Data {
    string result = 1;
}
```

### 第二步：生成代码 (Makefile)
使用 `protoc` 工具生成代码。

```makefile
gen:
	@protoc \
		--proto_path=protobuf "protobuf/service_name.proto" \
		--go_out=services/common/genproto/service_name --go_opt=paths=source_relative \
		--go-grpc_out=services/common/genproto/service_name --go-grpc_opt=paths=source_relative
```

### 第三步：服务端实现 (Server)

**1. 业务逻辑层 (Handler)**
路径: `services/service_name/handler/grpc.go`

```go
package handler

import (
	"context"
	"google.golang.org/grpc"
	pb "path/to/genproto/service_name" // 引入生成的包
)

// 定义 Handler 结构体，必须包含 UnimplementedMyServiceServer
type MyGrpcHandler struct {
	pb.UnimplementedMyServiceServer
    // 这里可以注入 Service 层依赖，例如：
    // service MyBusinessService
}

// 构造函数：注册服务
func NewGrpcMyService(grpcServer *grpc.Server) {
	handler := &MyGrpcHandler{}
	pb.RegisterMyServiceServer(grpcServer, handler)
}

// 实现接口方法
func (h *MyGrpcHandler) MyMethod(ctx context.Context, req *pb.MyRequest) (*pb.MyResponse, error) {
	// 1. 获取请求参数
	id := req.GetSomeId()
    
    // 2. 执行业务逻辑
    // result, err := h.service.Process(id)
    
    // 3. 返回响应
	return &pb.MyResponse{
		Success: true,
		Message: "OK",
        Data: &pb.Data{
            Result: "Processed " + id,
        },
	}, nil
}
```

**2. 启动服务 (Main)**
路径: `services/service_name/grpc.go`

```go
package main

import (
	"log"
	"net"
	"google.golang.org/grpc"
	handler "path/to/handler"
)

func RunGRPCServer(port string) error {
    // 1. 监听端口
	lis, err := net.Listen("tcp", port)
	if err != nil {
		return err
	}

    // 2. 创建 gRPC Server 实例
	server := grpc.NewServer()

    // 3. 注册服务
	handler.NewGrpcMyService(server)

	log.Printf("Starting gRPC server on %s", port)
    
    // 4. 启动服务
	return server.Serve(lis)
}
```

#### 第四步：客户端调用 (Client)
路径: `services/other_service/client.go`

```go
package main

import (
	"context"
	"log"
	"time"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
	pb "path/to/genproto/service_name"
)

func CallMyService() {
    // 1. 建立连接
    // 使用 insecure.NewCredentials() 因为是内部通信，未加密
	conn, err := grpc.NewClient("localhost:9000", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()

    // 2. 创建客户端 Stub
	client := pb.NewMyServiceClient(conn)

    // 3. 创建上下文 (建议设置超时)
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

    // 4. 发起调用
	res, err := client.MyMethod(ctx, &pb.MyRequest{
		SomeId: "123",
        Amount: 100,
	})
	if err != nil {
		log.Fatalf("call failed: %v", err)
	}

    // 5. 处理结果
	log.Printf("Response: %s", res.GetMessage())
}
```

## 3. 关键点总结

1.  **proto3**: 现代 gRPC 默认使用 proto3 语法
2.  **go_package**: 必须在 proto 文件中指定，决定生成代码的导入路径
3.  **Unimplemented...Server**: 在实现 Handler 时，必须嵌入这个结构体，这是 gRPC 生成代码的向前兼容性要求
4.  **Context**: 所有的 RPC 方法第一个参数都是 `context.Context`，用于控制超时和链路追踪
5.  **Listen vs Dial**: 服务端用 `net.Listen` + `Serve`，客户端用 `grpc.NewClient`

## 4. 深入理解 UnimplementedServer

在 gRPC Go 的生成代码中，强制要求嵌入 `Unimplemented<ServiceName>Server` 结构体。这是一个为了**向前兼容性**的设计

### 架构图解

```
  Interface: MyServiceServer
  +------------------------------------------+
  |  MethodA()                               |
  |  MethodB()                               |
  |  mustEmbedUnimplementedMyServiceServer() |
  +------------------------------------------+
                      ^
                      | implements
                      |
  Struct: UnimplementedMyServiceServer
  +------------------------------------------+
  |  MethodA() { return error }              |
  |  MethodB() { return error }              |
  |  mustEmbed...() {}                       |
  +------------------------------------------+
                      ^
                      | embedded in
                      |
  Struct: MyGrpcHandler (Your Implementation)
  +------------------------------------------+
  |  UnimplementedMyServiceServer  (Field)   | <--- Provides default MethodB & mustEmbed
  |                                          |
  |  MethodA() {                             | <--- OVERRIDES MethodA
  |     // Real business logic               |
  |     return success                       |
  |  }                                       |
  +------------------------------------------+
```

### 为什么必须嵌入？

1.  **接口实现的完整性**：
    *   生成的 `MyServiceServer` 接口可能包含 10 个方法
    *   如果你只想实现其中的 1 个方法，如果没有嵌入 `Unimplemented...`，Go 编译器会报错，因为它要求你实现接口的所有 10 个方法
    *   嵌入 `Unimplemented...` 后，它为你提供了剩余 9 个方法的默认实现（虽然是报错），让你只需关注你需要的方法

2.  **向前兼容性 (Forward Compatibility)**：
    *   **场景**：你在 `.proto` 文件中新增了一个 `NewMethod`
    *   **后果**：重新运行 `protoc` 后，生成的接口 `MyServiceServer` 会增加一个新方法
    *   **保护**：如果你的代码没有嵌入 `Unimplemented...`，你的代码会立即**编译失败**，因为你的 `MyGrpcHandler` 还没有实现 `NewMethod`
    *   **结果**：嵌入后，代码依然可以编译通过。调用新方法时，会由 `Unimplemented...` 里的默认实现处理，返回一个友好的 "Unimplemented" 错误，而不是导致服务崩溃或无法构建

3.  **必须嵌入的强制性 (`mustEmbed...`)**:
    *   生成的接口中包含一个名为 `mustEmbedUnimplemented<ServiceName>Server` 的私有方法
    *   这个方法只在 `Unimplemented<ServiceName>Server` 结构体中实现了
    *   **作用**：这意味着任何想要实现 `MyServiceServer` 接口的结构体，**必须**嵌入 `Unimplemented...` 结构体。如果你试图手动实现所有方法而不嵌入该结构体，由于缺少这个私有方法的实现，编译器会报错。这是 gRPC 官方为了强制实施向前兼容性策略而采用的一种技术手段

## 5. 生成文件详解

执行 `protoc` 命令后，会生成两个 Go 文件，它们职责明确，分工不同

| 文件名 | 生成参数 | 职责 | 内容详解 |
| :--- | :--- | :--- | :--- |
| **`*.pb.go`** | `--go_out` | **数据模型 (Model)** | 1. **消息结构体**: 对应 proto 中的 `message`，生成 Go struct<br>2. **序列化代码**: 实现 `proto.Message` 接口，包含 Marshal/Unmarshal 方法，负责二进制数据的序列化与反序列化<br>3. **辅助方法**: 如 `GetField()` 等 Getter 方法，用于安全访问字段（处理 nil 指针） |
| **`*_grpc.pb.go`** | `--go-grpc_out` | **通信层 (Transport)** | 1. **客户端接口 (Client)**: 定义了客户端调用的方法接口 (`New...Client`)，封装了底层的 gRPC 调用逻辑<br>2. **服务端接口 (Server)**: 定义了服务端需要实现的接口 (`...Server`)，包括 `Unimplemented...Server` 结构体<br>3. **注册函数**: 如 `Register...Server`，用于将你的实现注册到 gRPC Server 中 |

**简单记忆**：
*   `*.pb.go` = **数据** (Structs)
*   `*_grpc.pb.go` = **行为** (Interfaces & Methods)
