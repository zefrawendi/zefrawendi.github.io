---
title: "grpc"
date: 2022-10-26T11:45:48+08:00
lastmod: 2022-10-26T11:45:48+08:00
draft: false
description: "grpc实战：pc book"
categories: ["开发"]
tags: ["grpc"]
---

## 初始化项目

新建相关文件夹

```shell
mkdir -p pc-book/{proto,pb}

# proto 用于存放protobuf文件
# pb用于存放生成好的go代码
```

最终目录结构如下

```shell
.
├── README
├── go.mod
├── go.sum
├── main.go
├── pb
└── proto
```

## 编写proto文件

在proto文件夹下新建processor_message.proto

```protobuf
syntax = 'proto3';

// 需要加入下面的一行，
// option go_package = "path;name"
// path 表示生成的go文件的存放地址
// name 表示生成的go文件所属的包名
option go_package = "./;pb";

message CPU {
  string brand = 1;
  string name = 2;
  uint32 number_cores = 3;
  uint32 number_threads = 4;
  double min_ghz = 5;
  double max_ghz = 6;
}
```

## 配置grpc自动生成

### 1. 安装protoc

直接使用homebrew安装protoc

`brew install protobuf`

安装完成后就可以直接使用protoc命令了

### 2. 安装相关go-libraries

```shell
go get -u google.golang.org/grpc
go install github.com/golang/protobuf/protoc-gen-go
```

## 生成go代码

因为前面proto文件中已加入了option go_package = "path;name"，所以执行

`protoc --proto_path=proto proto/*.proto --go_out=plugins=grpc:pb`

即可在pb目录下生成对应的go代码：**processor_message.pb.go**

- –go_out：设置所生成 Go 代码输出的目录，该指令会加载 protoc-gen-go 插件达到生成 Go 代码的目的，生成的文件以 .pb.go 为文件后缀，在这里 “:”（冒号）号充当分隔符的作用，后跟命令所需要的参数集，在这里代表着要将所生成的 Go 代码输出到所指向 protoc 编译的当前目录。
- plugins=plugin1+plugin2：指定要加载的子插件列表，我们定义的 proto 文件是涉及了 RPC 服务的，而默认是不会生成 RPC 代码的，因此需要在 go_out 中给出 `plugins` 参数传递给 `protoc-gen-go`，告诉编译器，请支持 RPC（这里指定了内置的 grpc 插件）。
- --proto_path: 指定proto文件的路径

每次生成敲上面一串很麻烦，新建一个Makefile文件

```makefile
gen:
	protoc --proto_path=proto proto/*.proto --go_out=plugins=grpc:pb
	
clean:
	rm -fr pb/*.go
	
run:
	go run main.go
```

后续直接执行`make gen`、`make clean`、`make run`即可