---
title: autolayout
header: autolayout
description: autolayout
---

human life is ephemera, whick makes it precious.生命短暂，所以珍贵。

###Introduce TO Auto Layout

1. autolayout起源与cassowary,他是为解决用户界面布局问题而开发的。为了取代基于spring和strut的autosizing。与autosizing兼容。


2. 可用hasAmbiguousLayout测试视图约束是否充分。

3. 可视化约束，在OS X上调用visualizeConstraints:方法来可视化约束。

内在内容大小：视图内容大小通过每个视图的intrinsicContentSize属性表达。当改变了视图的内容大小时，需要调用invalidateIntrinsicContentSize，让autolayout知道在下次布局时重新计算。

4. 设置压缩阻力(1-1000),默认750,越大越不会剪切内容。

	[btn setContentCompressionResistancePriority:500 forAxis:UILayoutConstraintAxisVertical];
	
5. 设置内容吸附,默认250，越大超过内容大小的范围越小:

	[btn setContentHuggingPriority:500 forAxis:UILayoutConstraintAxisVertical];
	
6. 对齐矩形，在有装饰元素的视图上，只考虑矩形，而不考虑其他装饰性的元素。这些元素通常是直接绘制到图片上的，而不是通过layer或者view来添加的。
7. 可视化对齐矩形，在edit scheme->arguments中添加-UIViewShowAlignmentRects。在程序运行时，矩形会显示在各个视图上。
8. 对齐inset，要加上阴影效果分两步，第一步加载图像，第二步，在图像上调用imageWithAlignmentRectInsets来生成指定inset的新版本。

		//绘制到图片上占用内存少，运行效率高
		UIImage * image = [[UIImage imageNamed:@"Shadowed.png"] imageWithAlignmentRectInsets: UIEdgeInsetsMake(0, 0, 20, 20)];
		
这样手动构造inset有些麻烦。

###第二章 Contrainits

1. constrainit types - 
Layout constraints position and size.
2. content size constraints - self size related to content 就是压缩阻力(compression rules)和内容吸附(hugging rules)
3. Autosizing constraints - translate the older autoresizing masks into the auto layout system.
4. Layout support constraints 
5. Prototyping constraints

虽然只有第一个是公开的，但是我们可以通过公开的API和IB来创建所有的约束对象。我们在控制台经常能看到这些东西

	2013-07-17 09:56:26.788 HelloWorld[14733:c07] <NSAutoresizingMaskLayoutConstraint:0x767ae50 h=&-& v=&-& H:[UIView:0x7668030(30)]>	2013-07-17 09:56:26.789 HelloWorld[14733:c07] <NSLayoutConstraint:0x766bfc0 H:[UIImageView:0x766aac0(>=0)]>	2013-07-17 09:56:26.790 HelloWorld[14733:c07] <NSContentSizeLayoutConstraint:0x7674b00 H:[UIImageView:0x766aac0(512)] Hug:250 CompressionResistance:1>	2013-07-17 09:56:26.792 HelloWorld[14733:c07] <_UILayoutSupportConstraint:0x8e14e80 V:[_UILayoutGuide:0x8e1f260(0)]>	2013-07-17 09:56:26.793 HelloWorld[14733:c07] <NSIBPrototypingLayoutConstraint:0x895e390	'IB auto generated at build time for view with ambiguity' H:|-(137@251)-[UIButton:0x895b750](LTR) priority:251 (Names: '|':UIView:0x895b570 )>
	
在使用老的autosizing布局和自动布局的是混合系统中，会被翻译成相等的autosizing constraints，与auto layout共存。

content size constraints经常出现在label image 和 controls中，尤其是image。在自定义视图中表示自然大小。

在ib中，通过约束视图的顶部和底部来创建。在代码中，你的约束也许会涉及到控制器的topLayoutGuide或者bottomLayoutGuide属性。这些属性存储了UILayoutGuide的对象的引用。

举个例子：

	UIView * topLayoutGuide = (UIView *) self.topLayoutGuide;
	//然后addconstraints
	
>注意
>事实上，还有其他的内部约束类尤其在UIKit中，在日常工作中你并不会总是遇到window-anchoring、window-autoresizing或者scrollview 自动实际大小约束实例。他们也不会影响到我们建立自动约束。

优先级-priorities

优先级有系统定义的枚举

在运行时修改优先级可能会导致程序崩溃。