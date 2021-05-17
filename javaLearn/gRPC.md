# gRPC

![](./res/gRPC-packet-flow.png '')
![](./res/rpc-flow.jpg '')

![](./res/gRPC-netty-server-flow.jpg '')
### Service API IDL

```
// search.pb.go
var _SearchService_serviceDesc = grpc.ServiceDesc{
	ServiceName: "proto.SearchService",
	HandlerType: (*SearchServiceServer)(nil),
	Methods: []grpc.MethodDesc{
		{
			MethodName: "Search",
			Handler:    _SearchService_Search_Handler,
		},
	},
	Streams:  []grpc.StreamDesc{},
	Metadata: "search.proto",
}
```
>这看上去像服务的描述代码，用来向内部表述 “我” 都有什么。涉及如下:

ServiceName：服务名称
HandlerType：服务接口，用于检查用户提供的实现是否满足接口要求
Methods：一元方法集，注意结构内的 Handler 方法，其对应最终的 RPC 处理方法，在执行 RPC 方法的阶段会使用。
Streams：流式方法集
Metadata：元数据，是一个描述数据属性的东西。在这里主要是描述 SearchServiceServer 服务


### Register Service

```
func (s *Server) register(sd *ServiceDesc, ss interface{}) {
    ...
	srv := &service{
		server: ss,
		md:     make(map[string]*MethodDesc),
		sd:     make(map[string]*StreamDesc),
		mdata:  sd.Metadata,
	}
	for i := range sd.Methods {
		d := &sd.Methods[i]
		srv.md[d.MethodName] = d
	}
	for i := range sd.Streams {
		...
	}
	s.m[sd.ServiceName] = srv
}
```

>在最后一步中，我们会将先前的服务接口信息、服务描述信息给注册到内部 service 去，以便于后续实际调用的使用。涉及如下：

server：服务的接口信息
md：一元服务的 RPC 方法集
sd：流式服务的 RPC 方法集
mdata：metadata，元数据

### 总结
gRPC 基于 HTTP/2 + Protobuf。
gRPC 有四种调用方式，分别是一元、服务端/客户端流式、双向流式。
gRPC 的附加信息都会体现在 HEADERS 帧，数据在 DATA 帧上。
Client 请求若使用 grpc.Dial 默认是异步建立连接，当时状态为 Connecting。
Client 请求若需要同步则调用 WithBlock()，完成状态为 Ready。
Server 监听是循环等待连接，若没有则休眠，最大休眠时间 1s；若接收到新请求则起一个新的 goroutine 去处理。
grpc.ClientConn 不关闭连接，会导致 goroutine 和 Memory 等泄露。
任何内/外调用如果不加超时控制，会出现泄漏和客户端不断重试。
特定场景下，如果不对 grpc.ClientConn 加以调控，会影响调用。
拦截器如果不用 go-grpc-middleware 链式处理，会覆盖。
在选择 gRPC 的负载均衡模式时，需要谨慎。

* [gRPC-数据帧-偏底层](https://www.cnblogs.com/sunsky303/p/11119300.html)