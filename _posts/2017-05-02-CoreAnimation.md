---
title: CoreAnimation
description: CoreAnimation
header: CoreAnimation
---

The beauty of journey is found in the scenery along the way.

![](https://jeremy1221.github.io/img/CoreAnimation/coreanimation.png)

###Core Animation Manages Your APP's Content###

Core Animation is not a drawing system itself.It is an infrastructure for compositing and manipulating your app's content in hardware.At the heart of this infrastructure are *layer objects*, which you use to manage and manipulate your content.A layer captures your content into a bitmap that can be manipulated easily by the graphics hardware.In most apps, layers are used as a way to manage the content of views but you can also create standalone layers depending on your needs.

###Layer Modifications Trigger Animations

Most of the animations you crate using Core Animation involve the modification of the layer's properties.Like views, layer objects have a bounds rectangle, a position onscreen, an opacity, a transform, and many other visually-oriented properties that can be modified. For most of these properties, changing the propertie's value results in the creation of an implicit animation whereby the layer animates from the old value to the new value. You can also explicity animate these properties in cases where you want more control over the resulting animation behavior.

###Actions Let You Change a Layer's Default Behvaior

Implicit layer animations are archieved using action objects, which are geneirc objects that implement a predefined interface. Core Animation uses action obhects to implement the default set of animations normally associated with layers. You can create your own action objects to implement custom animations or use them to implement other types of behaviors too. You then assign your action object to one of the layer's properties. When that property changes, Core Animation retrieves your action object and tells it to perform its action.

###Layers Provide the Basis for Drawing and Animations

*Layer objects are 2D surfaces prganized in a 3D sa[ce and are at the heart pf everything you do with Core Animation. Like views, layers manage information about the geometry, content, and visual attributes of their surfaces. Unlike views, layers do not define their own appearance. A layer merely manages the state information surrounding a bitmap. The bitmap itself can be the result of a view drawing itself or a fixed image that you specify. For this reason, the main layers you use in your app are considered to be model objects because they primarily manage data. This is notion is important to remember because it affects the behavior of animations.

###The Layer-Based Drawing Model

Most layers do not do actual drawing in your app. Instead, a layer capture the content your app provides and caches it in a bitmap, which is sometimes referred to as the backing store. When you subsequently change a property of the layer, all you are doing is changing the state information associated with the layer object. When a change triggers an animation, Core Animation passes the layer's bitmap and state information to the graphics hareware, which does not the work of rendering the bitmap using the new information. Manipulating the bitmap is hardware in hareware yields much faster animations than could be done in software.

![](https://Jeremy1221.github.io/img/coreanimation/basics_layer_rendering.png)

Because it manipulates a static bitmap, layer-based drawing differs significantly from more traditional view-based drawing techniques. With view-based drawing, changes to the view itself often result in a call to view's drawRect: method to redraw content using the new parameters. But drawing in this way is expensive because it is done using the CPU on the main thread. Core Animation avoids this expense by whenever possible by manipulating the cached the same or similar effects.

Although Core Animation uses cached content as much as possible, your app must still provide the initial content and update it from time to time.

###Layer Trees Reflect Different Aspects of the Animation State

An App using Core Animation has three sets of layer objects. Each set of layer objects has a different role in making the content of your app appear onscreen:

* Objects in the *model laye tree*(or simply "layer tree") are the ones your app interacts with the most. The objects in this tree are the model objects that store the target values for any animations. Whenever you change the property of a layer, you use one of these objects.
* Objects in the presentation tree contain the in-fight values for any running animations, Whereas the layer tree objects contain the target values for an animation, the objects in the presentation tree reflect the current values as they appear on screen. You should never modify the objects in this tree, Instead, you use these objects tp read current animation values, perhaps to create a new aniumation starting at those values.
* Objects in the reder tree perform the actual animations and are private to Core Animation.

Each set of layer objects is organized into a hirarchical structure like the views in your app. In fact, for an app that enables layers for all of its viewsm the initial structure of each tree matches the structure of the view hierarchy exactly. Hpwever, an app can add additional layer objects-that is layers are not associated with a view-into the layer hierarchy as nedded. You might do this in situations to optimize your app's performancefor content that does not require all the overhead of a view. The window in the example contains a content view, which iteself contains a button view and two standalone layer objects. Each view has a corresponding layer object that forms part of the layer hierarchy.

![](https://jeremy1221.github.io/img/CoreAnimation/sublayer_hierarchy.png)

For every object in the layer tree, there is a matching object in the presentatiuon and render trees. As was previously mentioned, apps primarily work with objects in the layer tree but may at tines access obhects in the presentation tree. Specifically, accessing the pressentation:ayer property of an object in the layer tree returns the corresponding object in the persentation tree. You might want to access that object to read the current value of a property that is in the middle of an animation.

![](https://jeremy1221.github.io/img/CoreAnimation/sublayer_hierarchies.png)

> Important: You should access objects in the presentation tree only while an animation is in flgiht. While an animation is in progress, the presentation tree contains the layer values as they appear onscreen at that instant. This behavior differs from the layer tree, which always reflects the last value set by your code an is equivalent to the final state of the animation.


###The Relationship Between Layers and Views

Layers are not a replacement for your app's views-that is, your cannot create a visual interface based solely on layer objects. Layers provide infrastructure for your views. Specifically, layers make it easier and more efficient to draw and animate the contents of views and maintain high frame rates while doing so. However, there are many things that layers do not do. Layers do not handle events, draw content, participate in the responder chain, or do many other things. For this reason, every app must still have one or more views to handle those kinds of interactions.

In iOS, every view is backed by a corresponding layer object but in OS X you must decide which view should have layers. In OS X v10.8 and later, it probably makes sense to add layers to all of your views. However you are not required to do so and can still disalbe layers in cases where the overhead is unwarranted and unneeded. Layers do increase your app's memory overhead somewhat but their benefits often outweight the disadvantage, so it is always best to test the performance of your app before disabling layer support.

When you enable layers support for a view, you create what is referred to as a layer-backed view. In a layer-backed view, the system is responsible for creating the underlying layer object and for keeping that layer in sync with views. All iOS views are layer-backed and most views in OS X are as well.

> Note: For layer-backed views, it is recommended that you manipulate the view, rather that its layer, whenever possible. In iOS, views are just a thin wrapper around layer objects, so any manipulates you make to the layer usually work just fine. But there are case in both iOS and OS X where manipulating the layer instead of the view might not yield the desired results. Whenever possible, this document points out those pitfalls and tries to provided ways to help you work around them.

##Providing Layer's Contents

Layers are data objects that manage content provided by your app. A layer's content consists of a bitmap containing the visual data you want to display. You can provide the content for that bitmap in one of three ways:

* Assign an image object directly to the layer object's contents property.(This technique is best for layer content thath never, or raraly, changes.)
* Assign a delegate object to the layer and let the delegate draw the layer's content.(This technique is best for layer content that might change periodicallu and can be provided by an external object, such as a view.)
* Define a ;ayer subclass and override one of its drawing methods to provide the layer contents yourself.(This technique is appropriate if you have to create a custom layer subclass anyway or if you want to change the fundamental drawing behavior of the layer.)

The only time you need to worry aboyt providing content for a layer is when you create the layer object yourself. If your app contains nothing but layer-backed views, you do not have to worry about using any of the preceding techniqes to provide layer content. Layer-backed views automatically provide the contents for their associated layers in the most efficient way possible.

###速度控制函数(CAMediaTimingFunction)
1.kCAMediaTimingFunctionLinear（线性）：匀速，给你一个相对静态的感觉
2.kCAMediaTimingFunctionEaseIn（渐进）：动画缓慢进入，然后加速离开
3.kCAMediaTimingFunctionEaseOut（渐出）：动画全速进入，然后减速的到达目的地
4.kCAMediaTimingFunctionEaseInEaseOut（渐进渐出）：动画缓慢的进入，中间加

1. kCAFillModeRemoved 这个是默认值,也就是说当动画开始前和动画结束后,动画对layer都没有影响,动画结束后,layer会恢复到之前的状态 
2. kCAFillModeForwards 当动画结束后,layer会一直保持着动画最后的状态 
3. kCAFillModeBackwards 这个和kCAFillModeForwards是相对的,就是在动画开始前,你只要将动画加入了一个layer,layer便立即进入动画的初始状态并等待动画开始.你可以这样设定测试代码,将一个动画加入一个layer的时候延迟5秒执行.然后就会发现在动画没有开始的时候,只要动画被加入了layer,layer便处于动画初始状态 
4. kCAFillModeBoth 理解了上面两个,这个就很好理解了,这个其实就是上面两个的合成.动画加入后开始之前,layer便处于动画初始状态,动画结束后layer保持动画最后的状态.