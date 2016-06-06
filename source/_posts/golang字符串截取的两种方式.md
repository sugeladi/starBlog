title: golang字符串截取的两种方式
date: 2016-02-26 15:54:26
tags: [golang,小技巧]
categories: golang
---

打印出时间,发现采用lastIndex要比split的方式要快一些.


### 直接上代码
``` go
    package main
    
    import (
    "log"
    "strings"
    	"time"
    )
    
    func main()  {
    	name := "a.txh.doc"
    	t := time.Now()
    	for i:=0;i<1000;i++{
    		split(name)
    	}
    	log.Println(time.Now().Sub(t))
    
    	t1 := time.Now()
    	for i:=0;i<1000;i++{
    		lastIndex(name)
    	}
    	log.Println(time.Now().Sub(t1))
    }
```
<!--more-->
``` go    
    func split(name string) (int,string){
    	result := strings.Split(name, ".")
    	resultLen := len(result)
    	if resultLen < 2 {
    		return 200, "0"
    	}
    	filenameWithSuffixToLower := strings.ToLower(result[resultLen-1])
    	switch filenameWithSuffixToLower {
    	case "pdf":
    		return 200, "1"
    	case "doc", "docx":
    		return 200, "2"
    	case "xls", "xlsx":
    		return 200, "3"
    	case "ppt", "pptx", "pps":
    		return 200, "4"
    	case "png", "gif", "jpeg", "jpg", "bmp":
    		return 200, "5"
    	case "txt":
    		return 200, "6"
    	default:
    		return 200, "0"
    	}
    }
    
    func lastIndex(name string) (int,string){
    	index := strings.LastIndex(name,".")
    	if index < 0 {
    		return 200, "0"
    	}
    	result := name[index+1:]
    	filenameWithSuffixToLower := strings.ToLower(result)
    	switch filenameWithSuffixToLower {
    	case "pdf":
    		return 200, "1"
    	case "doc", "docx":
    		return 200, "2"
    	case "xls", "xlsx":
    		return 200, "3"
    	case "ppt", "pptx", "pps":
    		return 200, "4"
    	case "png", "gif", "jpeg", "jpg", "bmp":
    		return 200, "5"
    	case "txt":
    		return 200, "6"
    	default:
    		return 200, "0"
    	}
    }
```

split内部的实现可看到源码(如下),内部也是去做的for循环字符串截取分隔,所以lastIndex直接获取到索引,直接截取:
```go
// Generic split: splits after each instance of sep,
// including sepSave bytes of sep in the subarrays.
func genSplit(s, sep string, sepSave, n int) []string {
	if n == 0 {
		return nil
	}
	if sep == "" {
		return explode(s, n)
	}
	if n < 0 {
		n = Count(s, sep) + 1
	}
	c := sep[0]
	start := 0
	a := make([]string, n)
	na := 0
	for i := 0; i+len(sep) <= len(s) && na+1 < n; i++ {
		if s[i] == c && (len(sep) == 1 || s[i:i+len(sep)] == sep) {
			a[na] = s[start : i+sepSave]
			na++
			start = i + len(sep)
			i += len(sep) - 1
		}
	}
	a[na] = s[start:]
	return a[0 : na+1]
}
```