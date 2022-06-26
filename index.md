## protobuf安装和使用

---
title: protobuf安装和使用
date: 2022-06-06 14:08:14
tags:
---

# Portobuf安装

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


## 欢迎来到我的BlOG

---
title: 欢迎来到我的BlOG
date: 2022-06-06 14:08:14
tags:
---

之前自己在Godaddy上搭建过个人Blog，在上面也写过不少东西，但后来VPS到期，没有续费，导致数据全被清掉了，简直是血泪教训，千万别自己用VPS搭，不但花钱，还有数据丢失的风险。所以最好还是选择大一点的Saas Blog厂商，CSDN广告满天飞，页面展示也非常凌乱，放弃。于是最后还是选择了github来进行输出。毕竟Github背靠微软，应该不会倒吧。

OK，以上就算是本博客的处女开篇。
