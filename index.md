
# 【Golang标准库源码解析】net/http包的ListenAndServe()

首先，调起ListenAndServe()的方式有两种，一种是直接http.ListenAndServe传参调用，一种是先初始化出一个http.Server结构体，然后通过Server结构体绑定的ListenAndServe()进行调用

## 直接调用的形式

![image](https://user-images.githubusercontent.com/4004100/176989493-27aa62c5-60d2-47b8-bf4a-1377db41aa27.png)

## 通过Server.ListenAndServe()的形式

![image](https://user-images.githubusercontent.com/4004100/176989508-c8cb9a0c-696f-4f66-ad15-8c604679e6d6.png)

这两种形式最终都会调server.ListenAndServe()

server.ListenAndServe()里会初始化出一个net.Listener()

![image](https://user-images.githubusercontent.com/4004100/176989521-4fb36566-cb07-496b-9cdc-42ed7bcc5768.png)

internetSocket会调用socket()函数进行网络连接

socket会进行一系列系统调用，而最终返回一个fd

关于FD的注释描述如下

> // FD is a file descriptor. The net and os packages embed this type in
// a larger type representing a network connection or OS file.
> 

> FD 是一个文件描述符。 net 和 os 包中嵌入了这种类型
表示网络连接或 OS 文件的较大类型。
> 

然后将返回的TCPListener，也就是图中的ln变量传给sev.Serve()
![image](https://user-images.githubusercontent.com/4004100/176989534-a172ec99-eb8a-40f2-8c9c-f77a619bfbb0.png)


如上图所示，整个过程中，而最核心的方法是srv.Serve(),下面我们就针对srv.Serve()进行分析

```go
func (srv *Server) Serve(l net.Listener) error {
	if fn := testHookServerServe; fn != nil {
		fn(srv, l) // call hook with unwrapped listener
	}

	origListener := l
	l = &onceCloseListener{Listener: l}
	defer l.Close()

	if err := srv.setupHTTP2_Serve(); err != nil {
		return err
	}

	if !srv.trackListener(&l, true) {
		return ErrServerClosed
	}
	defer srv.trackListener(&l, false)

	baseCtx := context.Background()
	if srv.BaseContext != nil {
		baseCtx = srv.BaseContext(origListener)
		if baseCtx == nil {
			panic("BaseContext returned a nil context")
		}
	}

	var tempDelay time.Duration // how long to sleep on accept failure

	ctx := context.WithValue(baseCtx, ServerContextKey, srv)
	for {
		rw, err := l.Accept()
		if err != nil {
			select {
			case <-srv.getDoneChan():
				return ErrServerClosed
			default:
			}
			if ne, ok := err.(net.Error); ok && ne.Temporary() {
				if tempDelay == 0 {
					tempDelay = 5 * time.Millisecond
				} else {
					tempDelay *= 2
				}
				if max := 1 * time.Second; tempDelay > max {
					tempDelay = max
				}
				srv.logf("http: Accept error: %v; retrying in %v", err, tempDelay)
				time.Sleep(tempDelay)
				continue
			}
			return err
		}
		connCtx := ctx
		if cc := srv.ConnContext; cc != nil {
			connCtx = cc(connCtx, rw)
			if connCtx == nil {
				panic("ConnContext returned nil")
			}
		}
		tempDelay = 0
		c := srv.newConn(rw)
		c.setState(c.rwc, StateNew, runHooks) // before Serve can return
		go c.serve(connCtx)
	}
}
```

上面的代码是核心代码，在一个死循环中会调用net包里的Accept()接口，当无错误返回时，会获取上下文信息，然后将上下文传给c.serve()去执行并且是用协程的并行或并发方式执行。

## c.serve()

c.serve()方法比较长，后面我们出专门的文章来解析c.serve()

![image](https://user-images.githubusercontent.com/4004100/176989546-d6f081f2-e30a-4adf-8042-baba0cc84935.png)
![image](https://user-images.githubusercontent.com/4004100/176989551-4df7bd77-c947-48fb-b4fa-6664de18a562.png)

#【gRPC】 protobuf安装和使用

## Portobuf安装

[https://github.com/protocolbuffers/protobuf](https://github.com/protocolbuffers/protobuf)

[https://github.com/grpc-ecosystem/grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)

```bash
go install     github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway     github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2     google.golang.org/protobuf/cmd/protoc-gen-go     google.golang.org/grpc/cmd/protoc-gen-go-grpc
```

vscode安装vscode-proto3插件

新建一个xxx.proto文件，定义Protobuf消息体

```protobuf
syntax = "proto3";
package api_platform_ayzy_gin;
option go_package = "api_platform_ayzy_gin/proto/gen/go;hspb";

message Trip {
    string start = 1;       //第一个字段为start
    string end = 2;         //第二个字段为end
    int64 duration_sec = 3;
    int64 fee_cent =4;

}
```

新建目录

```protobuf
mkdir -p gen/go
```

在gen/go目录下生成hs.pb.go文件

```protobuf
protoc --go_out=paths=source_relative:gen/go .\hs.proto
```

```protobuf
package main

import (
	hspb "api_platform_ayzy_gin/cmd/proto/gen/go"
	"encoding/json"
	"fmt"
	"log"

	"google.golang.org/protobuf/proto"
)

func main() {
	hs := hspb.Hs{
		Start:       "abc",
		End:         "111",
		DurationSec: 3600,
		FeeCent:     10000,
	}
	fmt.Println("原始消息体：", &hs)

	b, err := proto.Marshal(&hs)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("消息的二进制流：%X\n", b)
	bbb, err := json.Marshal(&hs)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("二进制流转为json消息：%s\n", bbb)

	var hs2 hspb.Hs
	proto.Unmarshal(b, &hs2)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("解析二进制流为原始消息体：", &hs2)

	bb, err := json.Marshal(&hs2)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("原始消息体转为json消息体：%s\n", bb)

}
```


# 欢迎来到我的BlOG


之前自己在Godaddy上搭建过个人Blog，在上面也写过不少东西，但后来VPS到期，没有续费，导致数据全被清掉了，简直是血泪教训，千万别自己用VPS搭，不但花钱，还有数据丢失的风险。所以最好还是选择大一点的Saas Blog厂商，CSDN广告满天飞，页面展示也非常凌乱，放弃。于是最后还是选择了github来进行输出。毕竟Github背靠微软，应该不会倒吧。

OK，以上就算是本博客的处女开篇。
