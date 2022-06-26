# protobuf安装和使用

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
