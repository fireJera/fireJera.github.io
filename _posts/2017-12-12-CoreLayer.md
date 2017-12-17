---
title: core layer
description: core layer
header: core layer
---

why are you keeping curiosrity door locked.

CALayer 重绘图片有三种算法。
kCAFilterLinear 默认 采用双先性滤波算法
kCAFilterNearest 取最近的像素点，不管其他的颜色，适合直线。
kCAFilterTrilinear 三线性滤波算法，与双线性差不多。适合曲线比较多的图。
	
	view.layer.magnificationFilter = kCAFilterNearest;
	
组透明：
view-alpha
layer-opacity
对子层级都是有影响的。

设置info.plist中UIViewGroupOpacity为yes, 可以把整个图层树的透明效果设成一样的，但是这样做会影响整个APP。另一个方法，设置layer的shoulRasterize

矩阵仿射变换(Affine)-就是原图中的两条平行线，无论怎么变形，之后仍然是两条平行线。1\*3和3\*3矩阵相乘.
UIView.transform - CGAffineTransform (属性)
CALayer.transform - CATransform3D