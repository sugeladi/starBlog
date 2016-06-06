title: golang slice copy一个小坑
tags:
  - golang
categories: golang
date: 2016-04-06 16:41:33
---
今天遇到了一个slice在copy中的坑,在复制时一定要注意,目标slice的len是大于源slice的len.(不是cap)
直接上代码:
```go
    package main
    
    import "fmt"
    
    func main() {
    	srcSlice := []string{"hello","world"}
    	copy1 := []string{}
    	copy2 := make([]string, len(srcSlice))
    	copy(copy1, srcSlice)
    	copy(copy2, srcSlice)
    	fmt.Printf("srcSlice:%v \n",srcSlice)
    	fmt.Printf("copy1:%v \n",copy1)
    	fmt.Printf("copy2:%v \n",copy2)
    }

```

```shell
console:
    srcSlice:[hello world] 
    copy1:[] 
    copy2:[hello world] 
```