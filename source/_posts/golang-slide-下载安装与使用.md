title: golang .slide 下载安装与使用
tags:
  - golang
categories: golang
date: 2016-05-10 16:37:15
---
- 来源
    
    参加gopher china 2016的大会,在会上看到 [Dave](https://github.com/davecheney)大神用go的 slide 来做演讲,觉得好酷,根据[youtube](https://www.youtube.com/watch?v=83JBmS8WpHM)上的视频,然后自己跟着做了一遍,算是记录一下吧.

- 首先是下载

    由于GWF( Great FireWall 防火长城,也称中国防火墙 ),导致去go get golang.org/x/tools/cmd/present 下载不下来,这里推荐 [golangtc](http://www.golangtc.com/download/package),下载下来之后,放到你自己的gopath下

- 安装 present

    go install go/src/golang.org/x/tools/cmd/present/ 之后会在gopath的bin目录下多出来一个 present,
    
- 本地查看

    有了present就可以查看.slide文件了. 直接在有.slide文件的目录下,输入present,然后访问 http://127.0.0.1:3999/ 就可以看到啦.

- 编写.slide文件
<!--more-->
    
```shell
$ mkdir slide
$ cd slide
$ vi example.slide
```
    
    example.slide
    
    ```slide
    我的第一个Slide//标题
    练习一下//解释说明
    10 May 2016//时间 这里要注意一下格式
    
    苏格拉没有底//自己的名称
    http://star.asottt.com//自己的网站
    lihexingstar@163.com//邮箱
    @star//name
    
    * Formatting//文本格式
    
    Let's *bold* some text.//粗体使用 *我是粗体*
    Let's _italicize_ some text.//斜体使用 *我是斜体*
    
    * Images//图片
    
    .image http://7xk2rp.com1.z0.glb.clouddn.com/2011922123813232.jpg//图片外链
    
    * Showing Code//展示code(见一下 hello.go)
    
    .code hello.go//全部展示
    
    * Showing Partial Code//部分展示
    
    .code hello.go /show A type/,/end show A type/
    .code hello.go /start main OMIT/,/end main OMIT/
    .code hello.go /^func printStr/,/}/
    
    * Running code//运行代码
    
    .play hello.go /start main OMIT/,/end main OMIT/      //.play运行
    
    * Bullet Points//列表
    - bullet point 1
    - bullet point 2
    - bullet point 3
    ```
    
    hello.go
        
    ```golang
        package main
        
        import "fmt"
        
        //show A type OMIT
        type A struct{
            name string
        }
        //end show A type OMIT
        
        //start main OMIT
        func main(){
            fmt.Println("hello world. sugeladi")
        }
        //end main OMIT
        
        func printStr(s string){
            fmt.Println(s) // HL
        } 
    ```
    
- 外链放到go-talks(go开发者使用,第三方服务)和talks.godoc.org(golang官方服务) 需fanqiang看.

    你把自己编写的.slide文件放到github上,然后编写README
    ```README
    
    [go-talks](http://go-talks.appspot.com/github.com/sugeladi/slide/example.slide#1)
    
    [golang](http://talks.godoc.org/github.com/sugeladi/slide/example.slide)
    
    ```
- 效果如下:
    [go-talks](http://go-talks.appspot.com/github.com/sugeladi/slide/example.slide#1)
        
    [golang](http://talks.godoc.org/github.com/sugeladi/slide/example.slide)
    
    [kibana-go-talks](http://go-talks.appspot.com/github.com/sugeladi/slide/kibana.slide#1)
            
    [kibana-golang(推荐使用)](http://talks.godoc.org/github.com/sugeladi/slide/kibana.slide)
   
    