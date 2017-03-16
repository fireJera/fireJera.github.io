---
title: Runtime-Message Forwarding
description: How do i learn runtime
header: Runtime-Message Forwarding
---

The starting point of all achievement is desire.

Since swift has started to become mainstream development language on iOS, why still i study old runtime.Oh, fuck interview.

To begin with, I read this [blog](http://www.jianshu.com/p/db6dc23834e3)

The advantage of runtime:

* (1) Multiple Inheritance(实现多继承)
* (2) Method Swizzling
* (3) Aspect Oriented Programming
* (4) Isa Swizzling
* (5) Associated Object关联对象
* (6) class_addMethod(动态的增加方法)
* (7) NSCoding的自动归档和自动解档
* (8) 字典和模型互相转换

To find out every detail, I would like to read many article.[apple document](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-SW11)

However, there's an important difference between the two: Multiple inheritance combines different capabilities in a single object.It tends toward large, multifaceted objects. Forwarding, on the other hand, assigns separate responsibilities to disparate objects. It decomposes problems into smaller objects, but associates those objects in a way that's transparent to the message sender.

Although forwarding mimics inheritance, the NSObject class never confuses the two.Methods like respondsToSelector: and isKindOfClass: look only at the inheritance hierachy, never at the forwarding chain.

If u use forwarding to set up a surrogate object or to extend the capabilities of a class, the forwarding mechanism should probably be as transparent as inheritance. We can re-implement the respondToSelector: and isKindOfClassl methods to include your forwarding algorithm:

In addition to respondsToSelector: and isKindOfClass:, the instanceRespondToSelector: method should also mirror the forwarding algorithm. If protocols are used, the conformsToProtocol: method should likewise be added to the list.

Next, how to implement forwarding.There is 3 kind of ways.

first:

	#import <UIKit/UIKit.h>
	@interface Warrior : NSObject
	
	-(void)test;
	
	@end
	
	#import "Warrior.h"
	#import "Diplomat.h"
	#import <objc/runtime.h>
	
	@interface Warrior()
	
	@end
	
	@implementation Warrior
	
	void test(id self, SEL _cmd) {
	    NSLog(@"%@ %s", self, sel_getName(_cmd));
	}
	
	+(BOOL)resolveClassMethod:(SEL)sel {
	    NSLog(@"resolveClassMethod");
	    return YES;
	}
	
	+(BOOL)resolveInstanceMethod:(SEL)sel {
	    NSLog(@"resolveInstanceMethod");
	    if (sel == @selector(test)) {
	        class_addMethod(self, sel, (IMP)test, "v@:");
	        return YES;
	    }
	    return [super resolveInstanceMethod:sel];
	}
	@end
	
In this code, we have a class Warrior and didn't implement test method.but we can still invoke test method.

	Warrior * warrior = [[Warrior alloc] init];
	[warrior test];
	
second:

	-(id)forwardingTargetForSelector:(SEL)aSelector {
		if (aSelector == @selector(test)) {
			return [[Diplomat alloc] init];
		}
		return nil;
	}
	
third: 
	
	-(id)forwardingTargetForSelector:(SEL)aSelector {
    if (aSelector == @selector(test)) {
        return [[Diplomat alloc] init];
    }
    return nil;
	}
	
The difference between the three:

1. the three is called in order.
2. class_addMethod should be invoked in method 1
3. Method 2 only forward to 1 object.
4. Method 3 can forward to multiple objects.

If all the 3 didn't handle the message.Then throw err in the method doesNotRecognizeSelector.

	-(void)doesNotRecognizeSelector:(SEL)aSelector {
	    NSLog(@"doesNotRecognizeSelector");
	}
	
Method Swizzling:

I think it's relative simple, so I put id here.

	#import "UIViewController+Swizzling.h"
	#import <objc/runtime.h>
	
	@implementation UIViewController(Swizzling)
	
	+(void)load {
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        Class class = [self class];
	        //When swizzling a class method, use the following:
	        //Class class = object_getClass((id)self);
	        SEL originalSelector = @selector(viewWillAppear:);
	        SEL swizzlingSelector = @selector(myViewWillAppear:);
	        Method originalMethod = class_getInstanceMethod(class, originalSelector);
	        Method swizzlingMethod = class_getInstanceMethod(class, swizzlingSelector);
	        //judge the method will be swizlling is existed
	        BOOL didAddMethod = class_addMethod(class, originalSelector, method_getImplementation(swizzlingMethod), method_getTypeEncoding(swizzlingMethod));
	        if (didAddMethod) {
	            class_replaceMethod(class, swizzlingSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
	        } else {
	            method_exchangeImplementations(originalMethod, swizzlingMethod);
	        }
	    });
	}
	
	-(void)myViewWillAppear:(BOOL)animated {
	    [self myViewWillAppear:animated];
	}
	
	@end

>NOTE:
>
>1. swizzling should be invoked in load method.OC automatically calling two class method +load and +initialize in runtime.+load will be invoked when class initial load(会在初始加载时调用), +initialize will be called by lazy.If program don't send message to a class or its subclass, then +initialize will never be called. So we should put it in +load.
>
>2. swizzling should do in dispatch_once.Because double run will turn back original status before swizzling.
>
>3. Don't add [super load] in load, if there two class inherit the class.If they all do [super init], it will be the same as above.


swift method swizzling:

	public extension DispatchQueue {
   	private static var _onceTracker = [String]()
    
    /**
    Executes a block of code, associated with an unique token. The code is thread safe and will only execute the code once even in the presence of multithread calls.
     
     - parameter token: A unique reverse DNS style name such as com.vectorform.<name> or a GUID
     - parameter block: Block to execute once
    */
    
    public class func once(token: String, block: () -> Void) {
        objc_sync_enter(self)
        defer {
            objc_sync_exit(self)
        }
        if _onceTracker.contains(token) {
            return
        }
        
        _onceTracker.append(token)
        block()
    }
	}

	public extension DispatchQueue {
   	private static var _onceTracker = [String]()
    
    /**
    Executes a block of code, associated with an unique token. The code is thread safe and will only execute the code once even in the presence of multithread calls.
     
     - parameter token: A unique reverse DNS style name such as com.vectorform.<name> or a GUID
     - parameter block: Block to execute once
    */
    
    public class func once(token: String, block: () -> Void) {
        objc_sync_enter(self)
        defer {
            objc_sync_exit(self)
        }
        if _onceTracker.contains(token) {
            return
        }
        
        _onceTracker.append(token)
        block()
    }
}