---
title: objc_msgSend消息传递-消息转发
header: objc_msgSend消息传递-消息转发
description: objc_msgSend消息传递-消息转发
---

##The only way get smarter is by playing a smarter opponent.

只有与智者对弈，才能增长智慧。

##消息转发机制(message forwarding)

Objective-C在调用对象方法的时候，是通过消息传递机制来查询且执行方法。如果想令该类能够理解并执行方法，必须以程序代码实现出对应方法。但是在编译期间向类发送了无法解读的消息并不会报错，因为在runtime时期可以继续向类添加方法，所以编译器在编译时还无法确认类中是否已经实现了消息方法。

当对象接收到无法解读的消息后，就会启动消息转发机制，并且我们可以由此过程告诉对象应该如何处理位置消息。

本文的研究目标：当Class对象.h文件中声明了成员方法，但是没有对其进行实现，来跟踪一下runtime的消息转发过程，于是创造一下实验场景：

同上一篇文章一样，定义一个自定义Class Object，并且声明该Class中拥有方法-(void)test_no_exit，而在.m文件中不给予实现。在mian.m入口中直接调用该类某实例的-(void)test_no_exit方法。

![](https://Jeremy1221.github.io/img/msgSend/forward.jpg)

###动态方法解析

依旧在lookUpImpOrForward方法中下断点，并单步调试，观察代码走向。由于方法在方法列表中无法找到，所以立即进入method resolve过程。

	// 进入method resolve过程
	if (resolver && !triedResolver) {
		//释放读入锁
		runtimeLock.unlockRead();
		// 调用_class_resolveMethod,解析没有实现的方法
		_class_resolveMethod(cls, sel, inst);
		// 进行二次尝试
		triedResolve = YES;
		goto retry;
	}
	
runtimeLock.unlockRead()是释放读入锁操作，这里指缓存读入，即缓存机制不工作从而不会有缓存结果。随后进入_class_resolveMethod(cls, sel, inst)方法。

	void _class_resolveMethod(Class cls, SEL sel, id inst) {
		// 用isa查看是否指向元类 Meta Class
		if (! cls->isMetaCLass()) {
			// try [cls resolveInstanceMethod(cls, sel, inst)];
		} else {
			// try [nonMetaClass resolveClassMethod:sel]
			// and [cls resolveInstanceMethod:sel]
			_class_resolveClassMethod(cls, sel, inst);
			if (!lookUpImpOrNil(cls, sel, inst, NO/*initialize*/, YES/*cache*/, NO/*resolver*/)) {
			_class_resolveInstanceMethod(cls, sel, inst);
			}
		}
	}	
	
此方法是动态方法解析的入口，会间接地发送+resolveInstanceMethod或+resolveClassMethod消息。通过对isa指向的判断，从而分辨出如果是对象方法，则进入+resolveInstanceMethod方法，如果是类方法，则进入+resolveClassMethod方法。

而上述代码中的_class_resolveInstanceMethod方法，我们从源码中看到是如此定义的:

	static void _class_resolveInstanceMethod(Class cls, SEL sel, id inst) {
		if (! lookUpImpOrNil(cls->ISA(), SEL_resolveInstanceMethod, cls, NO/*initialize*/, YES/*cache*/, NO/*resolver*/)) {
			// Resolver not implemented.
			return;
		}
		// 构造布尔类型变量表达式，动态绑定函数
		BOOL (*msg)(Class, SEL, SEL) = (__typedof__(msg))objc_msgSend;
		// 获得是否重新传递消息标记
		bool resolved = msg(cls, SEL_resolveInstanceMethod, sel);
		// Cache the result (good or bad) so the resolver doesnot fire next time
		// +resolveInstanceMethod adds to self a.k.a cls
		// 调用lookUpImpOrNil 并重新启动缓存，查看是否已经添加上了选择子对应的IMP指针
		IMP imp = lookUpImpOrNil(cls, SEL, inst, NO/*initialize*/, YES/*cache*/, NO/*resolver*/);
		// 对查询到的IMP进行log输出
		if (resolved && PrintResolving) {
			if (imp) {
				_objc_inform("RESOLVE: method %c[%s %s] "
								"dynamically resolved to %p",
								cls->isMetaClass()? '+' : '-',
								cls->nameForLogging(), sel_getName(sel), imp);
			} else {
				// Method resolver didn't add anything?
				_objc_inform("RESOLVE: +[%s resolveInstanceMethod:%s] returned YES"
								", but no new implementation of %c[%s %s] was found",
								cls->nameForLogging(), sel_getName(self),
								cls->isMetaClass()? '+' : '-',
								cls->nameForLogging(), sel_getName(sel));
			}
		}
	}

通过_class_resolveInstanceMethod可以了解到，这只是通过+resolveInstabceMethod来查询是否开发者已经在云讯时将其动态插入类中实现函数。并且重新触发objc_msgSend方法。这里有一个C的语法值得我们去延伸学习一下，就是关键字__typeof__的。__typeof__(var)是GCC对C的一个扩展保留字([<font color=#f00>官方文档</font>](https://gcc.gnu.org/onlinedocs/gcc/Typeof.html))，这里是用来描述一个指针的类型。

我们发现，最终都会返回到objc_msgSend中。反观一下上一篇文章写的objc_msgSend函数，是通过汇编语言实现的。在[<font color=#f00>_Let's build objc\_msgSend_</font>](https://www.mikeash.com/pyblog/friday-qa-2012-11-16-lets-build-objc_msgsend.html)这篇资料中，记录一个关于objc_msgSend的伪代码。

	id objc_msgSend(id self, SEL, _cmd, ...) {
		Class c = object_getClass(self);
		IMP imp = cache_lookup(c, _cmd);
		if (!imp)
			imp = class_getMethodImplementation(c, _cmd);
			returm imp(self, _cmd, ...);
	}

在缓存中无法直接击中IMP时，会调用class_getMethodImplementation方法。在runtime中，查看一下class_getMethodImplementation方法。

	IMP class_getMethodImplementation(Class cls, SEL sel) {
		IMP imp;
		if (!cls || !sel) return nil;
		// 上一篇文章的搜索入口
		imp = lookUpImpOrNil(cls, sel, nil, YES/*initialize*/, YES/*cache*/, YES/*resolver*/);
		// Translate forwarding function to C-callable external version
		if (!imp) {
			return _objc_msgForward;
		}
		return imp;
	}
	
在上一篇文中，详细介绍过了lookUpImpOrNil函数成功搜索的流程。而本例中与前想反，我们发现该函数返回了一个_objc_msgForward的IMP。此时，我们击中的函数是_objc_msgForward这个IMP，于是消息转发机制进入了备援接收流程。

###Forwarding备援接收

_objc_msgForward 居然可以返回，说同IMP一样是一个指针。在_objc_msg_x86_64.s中发现了其汇编实现。

	ENTRY __objc_msgForward
	// Non-stret version
	// 调用 __objc_forward_handler
	movq	__objc_forward_handler(%rip), %r11
	jpm		*r11
	ENE_ENTRY		__objc_msgForward
	
发现在接收到_objc_msgForward指针后，会立即进入__objc_forward_handler方法。其源码在objc_runtime.mm中。

	#if !__OBJC2__
	// Default forward handler (nil) goes to forward:: dispatch.
	void * __objc_forward_handler = nil;
	void * __objc_forward_stret_handler = nil;
	#else
	// Default forward handler halts the process.
	__attribute_((noreturn)) void
	objc_defaultForwardHandler(id self, SEL sel) {
		_objc_fatal("%c[%s %s]: unrecognized selector sent to instance %p"
						"(no message forward handler is installed)",
						class_isMetaClass(object_getClass(self)) ? '+' : '-', 
						object_getClassName(self), sel_getName(sel), self);
	}
	void * _objc_forward_handler = (void *)objc_defaultForwardHandler;
	
在Objc2.0以前，_objc_forward_handler是nil，而在最新的runtime中，其实现由objc_defaultForwardingHandler完成。其源码仅仅是在log中记录一些相关信息，这也是handler的主要功能。

而抛开runtime，看见了关键字__attribute__((noreturn))。这里简单介绍了一下GCC中的又一扩展__attribute__机制。它用于与编译器直接交互，这是一个编译器指令(Compiler Directive)，用来在函数或数据声明中设置属性，从而进一步进行优化(继续了解可以阅读[<font color=#f00>_NShioster \_\_attribute\_\__</font>](http://nshipster.com/__attribute__/))。而在这里\_\_attribut\_\_((noreturn))是告诉编译器此函数不会返回给调用者，以编译器在优化时去掉不必要的函数返回代码。

Handler的全部工作是记录日志、触发crash机制。如果开发者想实现消息转发，则需要重写_objc_forward_handler中的实现。这时引入objc_setForwardHandler方法：

	void objc_setForwardHandler(void *fwd, void * fwd_stret) {
		_objc_forward_handler = fwd;
		#if SUPPORT_STRET
			_objc_forward_stret_handler = fwd_stret;
		#endif
	}

这是一个十分简单的动态绑定过程，让方法指针指向传入参数指针得以实现。

###Core Foundation 衔接

引入objc_setForwardHandler方法后，会有一个疑问:如何调用它？先来看一段异常信息:

	2016-08-27 08:26:08.264 debug-objc[7013:29381250] -[DGObject test_no_exist]: unrecognized selector sent to instance 0x101200310
	2016-08-27 10:09:16.495 debug-objc[7013:29381250] *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[DGObject test_no_exist]: unrecognized selector sent to instance 0x101200310'
	*** First throw call stack:
	(
		0   CoreFoundation                      0x00007fff842c64f2 __exceptionPreprocess + 178
		1   libobjc.A.dylib                     0x000000010002989f objc_exception_throw + 47
		2   CoreFoundation                      0x00007fff843301ad -[NSObject(NSObject) doesNotRecognizeSelector:] + 205
		3   CoreFoundation                      0x00007fff84236571 ___forwarding___ + 1009
		4   CoreFoundation                      0x00007fff842360f8 _CF_forwarding_prep_0 + 120
		5   debug-objc                          0x0000000100000e9e main + 94
		6   libdyld.dylib                       0x00007fff852a95ad start + 1
		7   ???                                 0x0000000000000001 0x0 + 1
	)
	libc++abi.dylib: terminating with uncaught exception of type NSException

这个日志场景都接触过。从调用栈上，发新了最终是通过Core Foundation抛出异常。在Core Foundation的CFRuntime.c无法找到objc_setForwardHandler方法的调用入口。综合参考[<font color=#f00>_Objective-C 消息发送与转发机制原理_</font>](http://yulingtianxia.com/blog/2016/06/15/Objective-C-Message-Sending-and-Forwarding/)和[<font color=#f00>_Hmmm，What's that Selector?_</font>](http://arigrant.com/blog/2013/12/13/a-selector-left-unhandled)两篇文章，我们发现了在[<font color=#f00>CFRuntime.c</font>](https://github.com/opensource-apple/CF/blob/master/CFRuntime.c)的__CFInitialize()方法中，实际上是调用了obcj_msgForwardHandler，这段代码被苹果公司隐藏。

在上述调用栈中，发现了Core Foundation中会调用__forwarding__。根据资料也可以了解到，在objc_setForwardHandler时会传入__CF_forwarding_prep_0和__forwarding_prep_1__两个参数，而这两个指针都会调用__forwarding__。这个函数中，也交代了消息转发的逻辑。在[<font color=f00>_Hmmm, What's that Selector?_</font>](http://arigrant.com/blog/2013/12/13/a-selector-left-unhandled)文章中，复原了__forwarding__的实现。

	// 两个参数：前者为被转发消息的栈指针 IMP， 后者为是否返回结构体
	int __forwarding__(void *frameStackPointer, int isStret) {
		id receiver = *(id *)frameStackPointer;
		SEL sel = *(SEL *)(framStackPointer + 8);
		const char *selName = sel_getName(sel);
		Class receiveClass = object_getClass(receiver);
		// 调用forwardingTargetForSelector;
		// 进入 备援接收 主要步骤
		if (class_respondsToSelector(receiverClass, @selector(forwardingTargetForSelector:))) {
			// 获得方法签名
			id forwardingTarget = [receiver forwardingTargetForSelector:sel];
			//判断返回类型是否正确
			if (forwardingTarget && forwarding != receiver) {
				//判断类型，是否返回值为结构体，选用不同的转发方法
				if (isStret == 1) {
					int rect;
					objc_msgSend_stret(&ret, forwardingTarget, sel, ...);
					return ret;
				}
				return objc_msgSend(forwardingTarget, sel, ...);
			}			
		}
		// 僵尸对象
		const char * className = class_getName(receiverClass);
		const char * zoombiePrefix = "_NSZombie_";
		size_t perfixLen = strlen(zombiePrefix, prefixLen); //0xa
		if (strncmp(className, zombiePrefix, prefixLen) == 0) {
			CFLog(KCFLogLevelError,
					@"*** -[%s %s]: message sent to deallocted instance %p",
					className + prefixLen,
					selName,
					receiver);
			<breakpoint-interrupt>
		}
		// 调用methodSignatureForSelector 获取方法签名后在调用 forwardInvocation
 		// 进入消息转发系统
 		if (class_responsToSelector(receiverClass, @selector(methodSignatureForSelector:))) {
 			NSMethodSignature * methodSignature = [receiver methodSignatureForSelector:sel];
 			// 判断返回类型是否正确
 			if (methodSignature) {
 				BOOL signatureIsStret = [methodSignature _frameDescriptor]->returnArgInfo.flags.isStruct;
 				if (signatureIsStret != isStret) {
 					CFLog(kCFLogLevelWarning,
 							@"*** NSForwarding : warning: method signature and compiler disagree on struct-return-edness of '%s'. Signature thinks it does%s return a struct, and compiler thinks it does%s.",
 							selName,
 							signatureIsStret ? "" : not,
 							isStret ? "" : not);
 				}
 				if (class_respondToSelector(receiverClass, @selector(forwardInvocation:))) {
 					// 传入消息的全部细节信息
 					NSInvocation * invocation = [NSInvocation _invocationWithMethodSignature:methodSignature frame:frameStackPointer]
 					[receiver forwardInvocation:invocation];
 					void *returnValue = NULL:
 					[invocation getReturnValue:&value];
 					return returnValue;
 				} else {
 					CFLog(KCFLogLevelWarning,
 							@"*** NSForwarding: warning: object %p of class '%s' does not implement forwardInvocation: -- dropping message",
 							receiver,
 							className);
 							return 0;
 				}
 			}
 		}
 		SEL * registeredSel = sel_getUid(selName);
		// selector 是否已经在 Runtime 注册过
		if (sel != registeredSel) {
			CFLog(kCFLogLevelWarning,
					@"*** NSForwarding: warning: selector (%p) for message '%s' does not match selector known to Objective-C runtime (%p)-- abort",
					sel,
					selName,
					registeredSel);
		}
		// doesNotRecognizeSelector, 主动抛出异常
		// 也就是前文我们看到的
		// 表明选择子未能得到处理
		else if (class_respondsToSelector(receiverClass, @selector(doesNotRecognizeSelector:))) {
			[receiver doesNotRecognizeSelector:sel];
		}
		else {
			CFLog(kCFLogLevelWarning,
					@"*** NSForwarding: warning: object %p of class '%s' does not implement doesNotRecognizeSelector: -- abort",
					receiver,
					className);
		}
		// The point of no return.
		kill(getpid(), 9);
	}
	
###Message-Dispatch System 消息派发系统

在大概了解过Message-Dispatch System 的源码后，来简单的说明一下。由于在前两步中，我们无法找到那条消息的实现。则创建一个NSInvocation对象，并将消息全部属性记录下来。NSInvocation对象包括了选择子、target以及其他参数。

随后，调用forwardInvocation:(NSInvoation *)invocation方法，其中的实现仅仅是改变了target指向，是消息保证能够调用。倘若发现本类无法处理，则继续向父类进行查找。直至NSObject，如果找到根类仍旧无法找到，则会调用doesNotRecognizeSelector:，以抛出异常。此异常表明选择子最终未能得到处理。

而对于doesNotRecognizeSelector:内部是如何实现，如何捕获异常。或者说override改方法后做自定义处理。


###对于消息转发的总结梳理

在Core Foundation的消息派发流程中，由于源码被隐藏，所以无法亲自测试代码， 倘若以后学习了逆向，可以再去探讨一下这里发生的过程。
对于这篇文章记录的消息转发流程，大致如下图所示：

![](https://Jeremy1221.github.io/img/msgSend/forwardFlow.png)

<table><tr><td bgcolor=#7FFFD4>
背景颜色
</td></tr></table>









	
	
