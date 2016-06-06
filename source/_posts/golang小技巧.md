title: golang小技巧
date: 2015-11-13 16:42:25
tags: [golang,小提示,小技巧]
categories: golang
---
### 小提示 

##### 对于这样的代码 ：
``` go
            file, err := os.Open("hx.txt")
            defer file.Close()
            if err != nil {
                ...  
                return
            }
```
<!--more-->
当文件 studygolang.txt 不存在或找不到时，file.Close()会panic，因为file是nil。因此，应该将defer file.Close()放在错误检查之后。
应该这样写：
``` go
            file, err := os.Open("hx.txt")
            if err != nil {
                ...
                return
            }
            defer file.Close()
```

