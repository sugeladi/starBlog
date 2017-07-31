title: glide 处理golang.org/x/net
tags:
  - glide
categories: glide
date: 2017-07-31 15:22:51
---
### 由于国内GWF得原因,导致使用glide引入golang.org/x 下得包的时候失败,经过查询,可以使用手动添加glide的cache来解决此问题

#### 进入glide的cacha目录
```golang
$ cd ~/.glide/cache/src/
$ mkdir https-golang.org-x-sys
$ cp -r xxx/* ./ 
$ glide install ##然后就可以愉快的下载啦
```
xxx代表自己已经下载好的源码 源码下载可以通过 [golangtc](https://www.golangtc.com/download/package) 进行下载.