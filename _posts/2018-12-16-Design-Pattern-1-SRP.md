---
title: Single Responsibility Principle
header: Single Responsibility Principle
description: Single Responsibility Principle
---

##Love looks not with the eyes, but with the mind.

Single Responsibility Principle 面向对象五(六)大原则之一
一个类(接口，方法)应该只有一个发生变化的原因。

Liskov Substitution Principle 里氏替换原则
第一种定义：
If for each object o1 of type S there is an object o2 of type T that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substituted for o2 then S is subtype of T

第二种定义：
Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.

有4层含义
1.子类必须完全实现父类的方法-----子类可以实现父类的方法，但是不能覆盖父类的非抽象方法。
2.子类可以有自己的个性
3.覆盖或实现父类的方法是参数必须是原参数的子类
4.返回结果必须是原参数的父类。


Dependence Inversion Principle 依赖倒置原则
High level modules should not depend upon low level modules. Both should depend upond abstractio. Abstraction should not depend upon details. Details should depend upon abstractions.