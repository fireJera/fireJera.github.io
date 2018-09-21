---
title: Function Programming
header: Function Programing
description: Function Programming
---

##It's never too old to learn.

###函数式编程

说到函数式编程就得提到另一种命令式编程。
命令式编程，以命令为主，给机器一条有一条的命令去执行，

函数式编程狭义上是没有变量的(没有可变的东西，所有的东西在定以后是不会发生改变的，即常量)

广义上，函数式编程意味着专注于函数。

	int fab(int n) {
	return n == 1 || n == 2 ? 1 : fab(n - 1) + fab(n - 2);
	}
	
这是用C语言写的求斐波那契数列的第N项的程序，相应的Haskell代码是这样的：

	fab :: (Num a) => a -> a
	fab n = if n == 1 || n == 2 then 1 else fab(n - 1) + fab(n - 2)
	
fab(5)
fab(4)
fab(3)
fab(2)
fab(1)
fab(2)
fab(3)
fab(2)
fab(1)

C语言会调用8次函数，

函数式语言的所有东西都是不变的，因此在执行的时候，过程是这样的

fab(5)
fab(4)
fab(3)
fab(2)
fab(1)
因为函数式编程的结果是唯一的，因此重复的计算可以用上次相同计算的缓存。

函数作为一等公民

函数式编程特性:
* 高阶函数
* 偏应用函数
* 柯里话
* 闭包

高阶函数就是参数或者返回值为函数的函数

闭包的基础是一等函数

[参考地址](https://blog.csdn.net/u013007900/article/details/79104110)

顺带一提响应式编程

响应式编程 用代码来解释吧

	a = 1
	b = 2
	c = b + a // c = 3
	
	a = 3 // c = 5
	b = 4 // c = 7
	
[自然要说到reactivCocoa](http://www.cocoachina.com/ios/20160729/17244.html)