title: 禁止ios的浏览器将连续数字识别为号码
tags:
  - [golang,templates,ios safari]
categories: html
date: 2016-05-06 16:56:05
---

今天在用go的templates做网页的时候,发现被ios的内置浏览用webview加载 会把连续的数字解析成一个可拨打的手机号,并带上链接显示蓝色的字体,在网上搜索了一下发下Safari的官网上说有个 meta 的属性如下
``` html
    <meta name="format-detection" content="telephone=no" />
```
记录一下~