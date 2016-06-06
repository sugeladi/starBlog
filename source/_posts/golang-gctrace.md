title: golang gctrace
tags:
  - golang
categories: golang
date: 2016-03-07 16:03:29
---
golang的GCTRACE(垃圾收集器追踪) 操作步骤:
### 1.先写点go代码,做点内存操作
test1.go :
```go
package main

import (
	"fmt"
	"log"
	"math/rand"
	"net/http"
)


type HXTest struct {
	Id        int
	Name string
	Email string
}
```
<!--more-->
```go
func main() {
	const SIZE = 1000000 // 100万
	m := make(map[int32]HXTest, SIZE)
	for i := 0; i < SIZE; i++ { // 把数据放进内存
		m[rand.Int31()] = HXTest{
			Id:i,
			Name:fmt.Sprintf("hx%d",i),
			Email:fmt.Sprintf("hx%d@gmail.com",i),
		}
	}

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// 模拟内存分配，做一些计算
		num := rand.Intn(10000)
		sum := 0
		for i := 0; i < num; i++ {
			sum = sum + i
		}
		fmt.Fprintf(w, "num:%d,sum: %d",num, sum)
	})
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### 2.运行起来,go build:mac的Terminal下:
```shell
go build -o gcTest test1.go
GODEBUG="gctrace=1" ./gcTest 2>log_file
```
说明:
将GOGCTRACE设置为1，Go程序就会在每次GC的时候输出GC的相关信息。
2>log_file 是把错误输出重定向到log_file的文件里.

### 3.使用wrk做测试: -d 10秒
```shell
wrk http://localhost:8080/ -d 10s
```
运行结果:
```text
hdeMacBook-Pro:goWeb lihexing$ wrk http://localhost:8080/ -d 10s 
Running 10s test @ http://localhost:8080/
  2 threads and 10 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    10.59ms   39.21ms 302.23ms   92.77%
    Req/Sec    29.45k     6.57k   36.60k    88.14%
  574902 requests in 10.02s, 75.79MB read
Requests/sec:  57371.48
Transfer/sec:      7.56MB
```

用tail -f log_file 查看文件如下:
```text
hdeMacBook-Pro:gcTest lihexing$ tail -f log_file 
gc 1 @0.007s 11%: 0.10+83+0.092 ms clock, 0.30+83/0.46/83+0.27 ms cpu, 92->92->92 MB, 93 MB goal, 8 P
gc 2 @0.109s 10%: 0.005+55+0.096 ms clock, 0.046+0.024/55/0.43+0.77 ms cpu, 93->93->93 MB, 184 MB goal, 8 P
gc 3 @0.819s 1%: 0.008+189+0.058 ms clock, 0.069+0.011/1.2/190+0.46 ms cpu, 139->139->132 MB, 186 MB goal, 8 P
gc 4 @7.581s 0%: 0.016+175+0.18 ms clock, 0.13+0.44/2.9/178+1.4 ms cpu, 231->231->139 MB, 265 MB goal, 8 P
gc 5 @9.555s 0%: 0.021+172+0.10 ms clock, 0.12+0.42/2.1/175+0.60 ms cpu, 261->261->139 MB, 279 MB goal, 8 P
gc 6 @11.755s 0%: 0.018+166+0.11 ms clock, 0.090+3.0/2.2/167+0.55 ms cpu, 270->270->139 MB, 279 MB goal, 8 P
gc 7 @14.022s 0%: 0.016+160+0.11 ms clock, 0.10+3.4/1.7/160+0.66 ms cpu, 272->272->139 MB, 279 MB goal, 8 P
GC forced
gc 8 @136.019s 0%: 0.023+191+0.071 ms clock, 0.18+0/3.3/189+0.57 ms cpu, 258->258->139 MB, 279 MB goal, 8 P
scvg0: inuse: 144, idle: 129, sys: 274, released: 0, consumed: 274 (MB)
GC forced
gc 9 @256.248s 0%: 0.018+225+0.17 ms clock, 0.14+0/4.1/223+1.4 ms cpu, 139->139->139 MB, 278 MB goal, 8 P
scvg1: inuse: 144, idle: 129, sys: 274, released: 0, consumed: 274 (MB)
GC forced
gc 10 @376.484s 0%: 0.011+182+0.10 ms clock, 0.091+0/3.2/180+0.86 ms cpu, 139->139->139 MB, 278 MB goal, 8 P
GC forced
gc 11 @496.675s 0%: 0.012+194+0.097 ms clock, 0.10+0/3.6/191+0.77 ms cpu, 139->139->139 MB, 278 MB goal, 8 P
scvg2: 129 MB released
scvg2: inuse: 144, idle: 129, sys: 274, released: 129, consumed: 144 (MB)
GC forced
gc 12 @616.890s 0%: 0.020+200+0.096 ms clock, 0.16+0/3.7/198+0.77 ms cpu, 139->139->139 MB, 278 MB goal, 8 P
scvg3: inuse: 144, idle: 129, sys: 274, released: 129, consumed: 144 (MB)
GC forced
gc 13 @737.102s 0%: 0.018+191+0.085 ms clock, 0.15+0/192/1.0+0.68 ms cpu, 139->139->139 MB, 278 MB goal, 8 P
```
