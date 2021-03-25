# 笔记

RPC(Remote Procedure Call)，是一个计算机通信协议。允许一台计算机上的程序 调用 另一台计算机上的子程序，而开发人员无需额外地关心这个交互过程。

服务A注册一个对象，并将 服务名以及方法名 暴露给服务B(暴露意味着服务B可以通过网络访问到)，服务B就可以像调用本服务的函数一样调用服务A的对象。

-   golang中实现RPC非常简单，官方提供了封装好的库，还有一些第三方的库
-   golang官方的net/rpc库使用encoding/gob进行编解码，支持tcp和http数据传输方式，由于其他语言不支持gob编解码方式，所以golang的RPC只支持golang开发的服务器与客户端之间的交互
-   官方还提供了net/rpc/jsonrpc库实现RPC方法，jsonrpc采用JSON进行数据编解码，因而支持跨语言调用，目前jsonrpc库是基于tcp协议实现的，暂不支持http传输方式

golang写RPC程序，必须符合4个基本条件，不然RPC用不了

-   结构体字段首字母要大写，即可导出
-   函数名必须首字母大写
-   函数有两个参数（类型可导出），一个返回值(error)。第一参数是接收参数，第二个参数是返回给客户端的参数，必须是指针类型，返回值必须是error类型
-   函数还必须有一个返回值error

形如：

```go
func (t *T) MethodName(argType T1, replyType *T2) error
```

其中 T1 和 T2 必须能够被 `encoding/gob` 序列化。

第一个参数T1是请求参数，T2 是返回参数，对于返回值 error，如果不为 `nil` ，最终底层传递的时候会被序列化成一个 `string`，效果类似于 `errors.New("...")`，此时 T2 并不会被传送给调用者。



一个范例：

服务端代码：

```go
package main

import (
	"errors"
	"fmt"
	"net/http"
	"net/rpc"
)

/*
* @CreateTime: 2020/10/10 10:27
* @Author: hujiaming
* @Description: 学习GRPC代码
 */

type Args struct {
	A, B int
}
type Quotient struct {
	Quo, Rem int
}

type Arith int

func (a *Arith) Multiply(req *Args, reply *int) error {
	*reply = req.A * req.B
	return nil
}
func (a *Arith) Divide(req *Args, reply *Quotient) error {
	if req.B == 0 {
		return errors.New("divide by zero")
	}
	reply.Quo = req.A / req.B
	reply.Rem = req.A % req.B
	return nil
}

// server端
func main() {
	arith := new(Arith)
    // 注册服务
	rpc.Register(arith)
    // 将服务处理绑定到 HTTP 上
	rpc.HandleHTTP()
	fmt.Println("grpc server start...")
    // 监听服务
	err := http.ListenAndServe(":8000", nil)
	if err != nil {
		panic(err)
	}
}
```

客户端代码：

```go
package main

import (
	"fmt"
	"net/rpc"
)

/*
* @CreateTime: 2020/10/10 11:21
* @Author: hujiaming
* @Description:
 */

var serverAddr = ""  // 服务端的地址

type Args struct {
	A, B int
}
type Quotient struct {
	Quo, Rem int
}
func main() {
    // 连接远程 RPC 服务
	conn, err := rpc.DialHTTP("tcp", serverAddr + ":8000")
	if err != nil {
		panic(err)
	}
	mul := 0
    // 调用 Multiply 方法
	if err := conn.Call("Arith.Multiply", Args{
		A: 20,
		B: 10,
	}, &mul); err != nil {
		panic(err)
	}
	fmt.Printf("调用multiply：%d\n", mul)

    // 调用 Divide 方法 
	div := &Quotient{}
	if err := conn.Call("Arith.Divide", Args{A: 20, B: 15}, div); err != nil {
		panic(err)
	}
	fmt.Printf("调用 divide: %d\n", div)
}
```

先启动server: `go run main.go` ，之后运行 client：`go run client.go`，可以看到结果：

```bash
调用multiply：200
调用 divide: &{1 5}
```





-   微服务架构下数据交互一般是对内 RPC，对外 REST
-   将业务按功能模块拆分到各个微服务，具有提高项目协作效率、降低模块耦合度、提高系统可用性等优点，但是开发门槛比较高，比如 RPC 框架的使用、后期的服务监控等工作
-   一般情况下，我们会将功能代码在本地直接调用，微服务架构下，我们需要将这个函数作为单独的服务运行，客户端通过网络调用



源码阅读顺序建议：[https://github.com/liangzhiyang/annotate-grpc-go](https://github.com/liangzhiyang/annotate-grpc-go)

