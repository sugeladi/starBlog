title: glide 处理golang.org/x/net
tags:
  - glide
categories: glide
date: 2017-07-31 15:22:51
---
### 由于国内GWF得原因,导致使用glide引入golang.org/x 下得包的时候失败,经过查询,可以使用手动添加glide的cache来解决此问题

#### 方式1使用mirror
```shell
$ rm -rf ~/.glide
$ mkdir -p ~/.glide
$ glide mirror set https://golang.org/x/mobile https://github.com/golang/mobile --vcs git
$ glide mirror set https://golang.org/x/crypto https://github.com/golang/crypto --vcs git
$ glide mirror set https://golang.org/x/net https://github.com/golang/net --vcs git
$ glide mirror set https://golang.org/x/tools https://github.com/golang/tools --vcs git
$ glide mirror set https://golang.org/x/text https://github.com/golang/text --vcs git
$ glide mirror set https://golang.org/x/image https://github.com/golang/image --vcs git
$ glide mirror set https://golang.org/x/sys https://github.com/golang/sys --vcs git
```

#### 方式2进入glide的cacha目录
```shell
$ cd ~/.glide/cache/src/
$ mkdir https-golang.org-x-sys
$ cp -r xxx/* ./ 
$ glide install ##然后就可以愉快的下载啦
```
xxx代表自己已经下载好的源码 源码下载可以通过 [golangtc](https://www.golangtc.com/download/package) 进行下载.