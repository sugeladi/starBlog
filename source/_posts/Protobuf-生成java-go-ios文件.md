title: Mac下安装Protobuf并生成java go ios文件
date: 2016-02-29 16:54:41
tags: [protobuf,golang,java,ios]
categories: protobuf
---

1.安装brew.查看本地是否 brew -v,如果没安装,执行以下代码 
```ruby
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
2.安装protobuf
```shell
brew install automake
brew install libtool
brew install protobuf
```
<!--more-->
3.制作java文件 执行 protoc -help 可以看到 java_out=OUT_DIR 直接就有java的,所以直接执行 
```shell
protoc --java_out=/Users/lihexing/Desktop/protobuf/java you.proto 
```
4.制作go文件,重点是插件 protoc-gen-go
    1. 下载源码 go get github.com/golang/protobuf/protoc-gen-go
    2. 进入目录 cd github.com/golang/protobuf/protoc-gen-go
    3. 编译插件 go install
    4. 使用插件 protoc --go_out=/Users/lihexing/Desktop/protobuf/go you.proto
5.制作ios文件 重点是插件 protobuf-objc
    1. 设置软连接 ln -s /usr/local/Cellar/protobuf/2.6.1/bin/protoc /usr/local/bin
    2. 克隆源码 git clone https://github.com/alexeyxo/protobuf-objc.git
    3. 进入目录 cd protobuf-objc
    4. 编译插件 ./scripts/build.sh
    5. 使用插件 pprotoc --plugin=/usr/local/bin/protoc-gen-objc kodec.proto --objc_out=/Users/lihexing/Desktop/protobuf/ios/
    
至此,在Mac下安装使用protobuf已完成.  至于如何写protobuf,还得接着学习~ 





执行protoc -help:
```shell
Usage: protoc [OPTION] PROTO_FILES
Parse PROTO_FILES and generate output based on the options given:
  -IPATH, --proto_path=PATH   Specify the directory in which to search for
                              imports.  May be specified multiple times;
                              directories will be searched in order.  If not
                              given, the current working directory is used.
  --version                   Show version info and exit.
  -h, --help                  Show this text and exit.
  --encode=MESSAGE_TYPE       Read a text-format message of the given type
                              from standard input and write it in binary
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
  --decode=MESSAGE_TYPE       Read a binary message of the given type from
                              standard input and write it in text format
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
  --decode_raw                Read an arbitrary protocol message from
                              standard input and write the raw tag/value
                              pairs in text format to standard output.  No
                              PROTO_FILES should be given when using this
                              flag.
  -oFILE,                     Writes a FileDescriptorSet (a protocol buffer,
    --descriptor_set_out=FILE defined in descriptor.proto) containing all of
                              the input files to FILE.
  --include_imports           When using --descriptor_set_out, also include
                              all dependencies of the input files in the
                              set, so that the set is self-contained.
  --include_source_info       When using --descriptor_set_out, do not strip
                              SourceCodeInfo from the FileDescriptorProto.
                              This results in vastly larger descriptors that
                              include information about the original
                              location of each decl in the source file as
                              well as surrounding comments.
  --error_format=FORMAT       Set the format in which to print errors.
                              FORMAT may be 'gcc' (the default) or 'msvs'
                              (Microsoft Visual Studio format).
  --print_free_field_numbers  Print the free field numbers of the messages
                              defined in the given proto files. Groups share
                              the same field number space with the parent 
                              message. Extension ranges are counted as 
                              occupied fields numbers.
  --plugin=EXECUTABLE         Specifies a plugin executable to use.
                              Normally, protoc searches the PATH for
                              plugins, but you may specify additional
                              executables not in the path using this flag.
                              Additionally, EXECUTABLE may be of the form
                              NAME=PATH, in which case the given plugin name
                              is mapped to the given executable even if
                              the executable's own name differs.
  --cpp_out=OUT_DIR           Generate C++ header and source.
  --java_out=OUT_DIR          Generate Java source file.
  --python_out=OUT_DIR        Generate Python source file.
```