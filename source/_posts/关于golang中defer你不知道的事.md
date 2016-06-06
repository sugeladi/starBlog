title: 关于golang中defer你不知道的事
date: 2015-10-15 11:23:28
tags: [golang,defer]
categories: golang
---
文章出处:<http://www.xiaozhou.net/something-about-defer-2014-05-25.html>


### Golang中的defer关键字实现比较特殊的功能，按照官方的解释，defer后面的表达式会被放入一个列表中，在当前方法返回的时候，列表中的表达式就会被执行。一个方法中可以在一个或者多个地方使用defer表达式，这也是前面提到的，为什么需要用一个列表来保存这些表达式。在Golang中，defer表达式通常用来处理一些清理和释放资源的操作
<!--more-->
---
# defer表达式中变量的值在defer表达式被定义时就已经明确
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}

### 上面的这段代码，defer表达式中用到了i这个变量，i在初始化之后的值为0，接着程序执行到defer表达式这一行，表达式所用到的i的值就为0了，接着，表达式被放入list，等待在return的时候被调用。所以，后面尽管有一个i++语句，仍然不能改变表达式 fmt.Println(i)的结果。所以，程序运行结束的时候，输出的结果是0而不是1。
---
# defer表达式的调用顺序是按照先进后出的方式
func b() {
    defer fmt.Print(1)
    defer fmt.Print(2)
    defer fmt.Print(3)
    defer fmt.Print(4)
}
### defer表达式会被放入一个类似于栈(stack)的结构，所以调用的顺序是后进先出的。所以，上面这段代码输出的结果是4321而不是1234。在实际的编码中应该主意，程序后面的defer表达式会被优先执行。
---
# defer表达式中可以修改函数中的命名返回值
func c() (i int) {
    defer func() { i++ }()
    return 1
}
### 上面的示例程序，返回值变量名为i，在defer表达式中可以修改这个变量的值。所以，虽然在return的时候给返回值赋值为1，后来defer修改了这个值，让i自增了1，所以，函数的返回值是2而不是1。