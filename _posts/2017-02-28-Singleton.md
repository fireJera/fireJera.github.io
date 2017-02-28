---
title: Singleton
description: This is an introduction to singleton in ios.
header: singleton
---

昨天去面试遇到了一些平时很常见但是从没花过心思的题，然后回来整理了一下，没想到万恶的csdn把我写了一下午的心得根本没有保存，只留下一行寥寥无几的文字在那赤裸裸的嘲讽我。

言归正传，下面首先讲一讲singleton的写法：

OC版：

1.

	@implementation Singleton
	static Singleton *instance = nil;
	
	+(instancetype) sharedInstance {
	    @synchronized (self) {
	        if (instance == nil) {
	            instance = [[Singleton alloc] init];
	            return instance;
	        }
	    }
	    return instance;
	}
	
	+(instancetype) allocWithZone:(struct _NSZone *)zone {
	    @synchronized(self) {
	        if (instance == nil) {
	            instance = [super allocWithZone: zone];
	            return instance;
	        }
	    }
	    return instance;
	}
	
	-(id)copyWithZone:(NSZone *)zone {
	    return self;
	}

2.

	@implementation Singleton
	static Singleton *instance = nil;
	
	+(instancetype)sharedInstance {
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        if (instance == nil) {
	            instance = [[Singleton alloc] init];
	        }
	    });
	    return instance;
	}
	
	+(instancetype)allocWithZone:(struct _NSZone *)zone {
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        if (instance == nil) {
	            instance = [super allocWithZone:zone];
	        }
	    });
	    return instance;
	}
	
	-(id)copyWithZone:(NSZone *)zone {
	    return instance;
	}
	
	-(id)mutableCopyWithZone:(NSZone *)zone {
	    return instance;
	}
看见这些我心里有很多疑问，@synchronized是个嘛东西，原理？dispatch\_once又是干嘛的，allocWithZone又是什么？dispatch\_once\_t？

首先我们来看看@synchronized和dispatch\_once，先举个例子，比如现在同时有两个线程(A和B)要使用sharedInstance创建实例，当两个线程同事运行到@synchronized的代码块时，其中一个线程(A)会上锁，然后线程B会进入睡眠直到线程A运行结束，当线程A运行结束时instance已经不是nil了，所以线程B不会再创建。同理，dispatch\_once会保证这段代码只运行一次，所以线程A先运行后，线程B就不会再运行了。[原文戳这里](http://www.cocoachina.com/ios/20160613/16661.html)

下面我们来看看dispatch\_once\_t与dispatch\_once:

!["dispatch_once_t"]()

To help resolve these issues, [The Review Index](https://thereviewindex.com) has launched its 

The public beta was launched in January 2017 with six product categories, which include [Mobiles](https://thereviewindex.com/Mobiles), [Speakers](https://thereviewindex.com/speakers), [Televisions](https://thereviewindex.com/televisions), [Routers](https://thereviewindex.com/routers), [Microwaves](https://thereviewindex.com/microwaves) and [Washing Machines](https://thereviewindex.com/washingmachines) and caters to Indian consumers. Reviews from popular online stores are compiled and reconciled using Neural Network based algorithms, and feature-wise summaries of the aggregated opinion is created.



