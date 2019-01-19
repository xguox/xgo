---
title: "Protobuf, Go, gRPC 的 Hello World 理解"
date: 2018-08-10T16:01:23+08:00
draft: false
categories: ["Protobuf", "Go", "RPC"]
tags: ["Protobuf", "Go", "RPC"]
---


[示例代码](https://github.com/xguox/protobuf_example)从[官方示例](https://github.com/google/protobuf/tree/master/examples)提取过来的.

```go
// addressbook.proto

syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto"; // 导入 proto3 新加的 timestamp 包
```

对于目标语言为 Go, 除非显式的提供 `option go_package` 在 **.proto** 文件中, 否则, 如果指定了 package, 则编译后就是对应的 Go 文件所属的 package 了. 不过官方建议, 不管是否定义了 `option go_package`, 又或者是 Go 之外其他语言也最好指定这个 package, 这样可以用于防止不同的消息类型有命名冲突.

#### The Protocol Buffer API

protoc 编译生成的 `addressbook.pb.go` 提供了下面这些有用的类型:

- An AddressBook structure with a People field.
- A Person structure with fields for Name, Id, Email and Phones.
- A Person_PhoneNumber structure, with fields for Number and Type.
- The type Person_PhoneType and a value defined for each value in the Person.PhoneType enum.

##### Writing a Message

**Protocol Buffers** 的出现就是为了序列化数据使其能在其他地方使用. 对于 Go 来说, 序列化数据用的是 `proto` 库的 [`Marshal` 函数](https://godoc.org/github.com/golang/protobuf/proto#Marshal). 对应于 `.proto` 中的 Message, 在 Go 的呈现是一个指向实现了 `proto.Message` 接口的结构体的指针. 通过调用 `proto.Marshal` 得到编码成二进制格式的数据(`wire format`). 比如这里的 [add_person](https://github.com/xguox/protobuf_example/blob/master/add_person.go)

```go
book := &pb.AddressBook{}
// ...

// Write the new address book back to disk.
out, err := proto.Marshal(book)
if err != nil {
    log.Fatalln("Failed to encode address book:", err)
}
if err := ioutil.WriteFile(fname, out, 0644); err != nil {
    log.Fatalln("Failed to write address book:", err)
}
```

##### Reading a Message

当需要解码数据的时候, 则用的是 `proto` 库的 [`Unmarshal` 函数](https://godoc.org/github.com/golang/protobuf/proto#Unmarshal), 比如 [list_people](https://github.com/xguox/protobuf_example/blob/master/list_people.go) 这里从上面的写入文件中读取数据并序列化为 Go 的结构体(`book pb.AddressBook`).

```go
// Read the existing address book.
in, err := ioutil.ReadFile(fname)
if err != nil {
    log.Fatalln("Error reading file:", err)
}
book := &pb.AddressBook{}
if err := proto.Unmarshal(in, book); err != nil {
    log.Fatalln("Failed to parse address book:", err)
}
```

##### Extending a Protocol Buffer

如果希望在更新了 `.proto` 能 **向后兼容**, 那么在新版本的 `.proto` 中需要做到以下几点:

- 所有已存在的字段的唯一标识号不能改变
- 可以删除字段, 但是请看第三条
- 可以添加字段, 但需要使用全新的标识号(从未用过的, **已经用过但字段被删除了也不可以**)

-------------------

### Hello World, gRPC

#### Server

示例代码在安装 [gRPC](https://github.com/grpc/grpc-go) `go get -u google.golang.org/grpc` 的时候就已经有了的,

```go
cd $GOPATH/src/google.golang.org/grpc/examples/helloworld

go run greeter_server/main.go

// From a different terminal:

go run greeter_client/main.go

=> 2018/08/09 19:59:19 Greeting: Hello world
```

**hello.proto**

如果想要将 Message 用在 **RPC(远程过程调用)**系统中, 可以在 `.proto` 文件中定义一个 rpc 服务接口, Protocol Buffer 编译器会像 `message` 一样编译成对应语言的代码. 比如下面的 `helloworld.proto`, 定义一个 rpc 方法, 接收 `HelloRequest` 消息并返回一个 `HelloReply` 消息:

```go
// The greeting service definition.
service Greeter {
  // Sends a greeting
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
    string name = 1;
}

// The response message containing the greetings
message HelloReply {
    string message = 1;
}
```

编译成 Go 以后是这里的 [helloworld.pb.go](https://github.com/xguox/grpc_example/blob/master/helloworld/helloworld.pb.go). 相比起前半部分的例子, 主要区别是这个

```go
service Greeter {
    // Sends a greeting
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}
```

对应的, 生成的 Go 代码主要多了两个接口:

```go
// GreeterClient is the client API for Greeter service.
//
// For semantics around ctx use and closing/ending streaming RPCs, please refer to https://godoc.org/google.golang.org/grpc#ClientConn.NewStream.
type GreeterClient interface {
    // Sends a greeting
    SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error)
}

type greeterClient struct {
    cc *grpc.ClientConn
}

func NewGreeterClient(cc *grpc.ClientConn) GreeterClient {
    return &greeterClient{cc}
}

func (c *greeterClient) SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error) {
    out := new(HelloReply)
    err := c.cc.Invoke(ctx, "/helloworld.Greeter/SayHello", in, out, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}

// GreeterServer is the server API for Greeter service.
type GreeterServer interface {
    // Sends a greeting
    SayHello(context.Context, *HelloRequest) (*HelloReply, error)
}

```

server 端 `main.go`

这里的 pb 是 helloworld 包的别名

```go
// server is used to implement helloworld.GreeterServer.
type server struct{}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
    return &pb.HelloReply{Message: "Hello " + in.Name}, nil
}

```

`lis, err := net.Listen("tcp", port)` 绑定端口，然后监听，当有消息到来时，读取并回复给客户端

Go 的 `net` 包提供了网络 I/O(TCP/IP, UDP, domain name resolution, Unix domain sockets) 需要用到的接口

![](http://wx1.sinaimg.cn/large/62fdd4d5gy1fu3p2zidinj225m0tctha.jpg)


```go
    s := grpc.NewServer()
```

![](http://wx3.sinaimg.cn/large/62fdd4d5gy1fu3p9qakuhj214k0angnx.jpg)

```go
    pb.RegisterGreeterServer(s, &server{})
    // Register reflection service on gRPC server.
    reflection.Register(s)
```

**helloworld.pb.go** 的函数 `RegisterGreeterServer`, 实际上调的是 grpc.Server 的实例方法 `RegisterService`,

`_Greeter_serviceDesc` 看名字就是对这个 **Greeter** rpc 服务的一些描述了,

```go
var _Greeter_serviceDesc = grpc.ServiceDesc{
    ServiceName: "helloworld.Greeter",
    HandlerType: (*GreeterServer)(nil),
    Methods: []grpc.MethodDesc{
        {
            MethodName: "SayHello",
            Handler:    _Greeter_SayHello_Handler,
        },
    },
    Streams:  []grpc.StreamDesc{},
    Metadata: "helloworld.proto",
}

...

func RegisterGreeterServer(s *grpc.Server, srv GreeterServer) {
    s.RegisterService(&_Greeter_serviceDesc, srv)
}
```

![](http://wx4.sinaimg.cn/large/62fdd4d5gy1fu3pp8x0tdj21li0l6wjp.jpg)

`ht := reflect.TypeOf(sd.HandlerType).Elem()` 这里也就是 `reflect.TypeOf((*pb.GreeterServer)(nil)).Elem()` 结果是 `helloworld.GreeterServer`

`st := reflect.TypeOf(ss)` 对应的是 `reflect.TypeOf(&server{})`, 返回结果是 `*main.server`

也就是我们在 **main.go** 的 `server` 结构体实现了 `helloworld.GreeterServer` 这个接口, 所以, `st.Implements(ht)` 返回 `true`

深入下去就不够 hello world 了, 往后看看再深入分析一下, 总的来说就是注册这个服务呗.

最后这里 `s.Serve(lis)`, 本来想贴源码的, 差不多一百行算了, 贴描述好了

> Serve accepts incoming connections on the listener lis, creating a new
    ServerTransport and service goroutine for each. The service goroutines
    read gRPC requests and then call the registered handlers to reply to them.
    Serve returns when lis.Accept fails with fatal errors.  lis will be closed when
    this method returns.
    Serve will return a non-nil error unless Stop or GracefulStop is called.

#### Client

主要代码就下面这三句:

```go
// Set up a connection to the server.
conn, err := grpc.Dial(address, grpc.WithInsecure())

c := pb.NewGreeterClient(conn)

r, err := c.SayHello(ctx, &pb.HelloRequest{Name: name})

```

```go
// Dial creates a client connection to the given target.
func Dial(target string, opts ...DialOption) (*ClientConn, error) {
    return DialContext(context.Background(), target, opts...)
}

// DialContext creates a client connection to the given target. By default, it's
// a non-blocking dial (the function won't wait for connections to be
// established, and connecting happens in the background). To make it a blocking
// dial, use WithBlock() dial option.
//
// In the non-blocking case, the ctx does not act against the connection. It
// only controls the setup steps.
func DialContext(ctx context.Context, target string, opts ...DialOption) (conn *ClientConn, err error) {
    ...
}

// ClientConn represents a client connection to an RPC server.
type ClientConn struct {
    ...
}
```

**helloworld.pb.go**

```go
type greeterClient struct {
    cc *grpc.ClientConn
}

func NewGreeterClient(cc *grpc.ClientConn) GreeterClient {
    return &greeterClient{cc}
}
```

```
r, err := c.SayHello(ctx, &pb.HelloRequest{Name: name})
...
log.Printf("Greeting: %s", r.Message)
```

对应的 `SayHello` 方法和 `HelloReply` 结构体

```go
// The response message containing the greetings
type HelloReply struct {
    Message              string   `protobuf:"bytes,1,opt,name=message,proto3" json:"message,omitempty"`
    XXX_NoUnkeyedLiteral struct{} `json:"-"`
    XXX_unrecognized     []byte   `json:"-"`
    XXX_sizecache        int32    `json:"-"`
}

func (c *greeterClient) SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloReply, error) {
    out := new(HelloReply)
    err := c.cc.Invoke(ctx, "/helloworld.Greeter/SayHello", in, out, opts...)
    if err != nil {
        return nil, err
    }
    return out, nil
}
```
