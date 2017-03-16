---
title: runtime-Aspect Oriented Programming
description: runtime-Aspect Oriented Programming comprehension
header: runtime-Aspect Oriented Programming
---

Adventure may hurt you, but monotony will kill you.

message: selector and argument
selector: name of method
method: selector + imp
implemention: just body of method
SEL:根据方法名生成一个来区分方法的唯一的ID，只要名字(包括参数)相同，SEL就是一样的。
IMP:给方法起一个名字，直接拿到函数的地址指针，通过指针直接调用，不用通过对象来调用。
CLASS: