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

Simplicity is ultimate sophistication(简约才是精致到了极致)
四：isa-swizzling
添加kvo之后，isa指针发生了变化。由Student变成->NSKVONotifying_Student,在执行完addObserver: forKeyPath: options: context: 之后，把isa指向到另外一个类去。
在新类里面会重写被观察的对象的四个方法：class、setter、delloac、_isKVOA。

1. 重写class方法： 为了保证返回与重写之前一样的内容。
2. 重写setter方法： 为了增加两个方法

	\- (void)willChangeValueForKey:(NSString *)key
	
	\- (void)didChangeValueForKey:(NSString *)key
	
在didChangeValueForKey:会调用
	
	- (void)observeValueForKeyPath:(NSString *)keyPath
	                      ofObject:(id)object
	                        change:(NSDictionary *)change
	                       context:(void *)context

3. 重写delloac方法： 释放生成的NSKVONotifying_类。
4. 重写_isKVOA方法。

>注意
>永远不要用用isa来判断一个类的继承关系，而是应该用class方法来判断类的实例。
>KVO缺陷:不能用selector或block是写回调，只能重写-addObserver:forKeyPath:options:context:，而且如果观察的属性多了需要添加多个if-else来判断。

五：Associated Object:
category中无法直接添加@property，因为添加之后并不会自动生成实例变量和存取方法。可通过关联对象来添加属性。

	@interface NSObject (NSObject_AssociatedObject)
	@property(nonatomic, strong) id associatedObject;
	@end
	
	@implementation NSObject (NSObject_AssociatedObject)
	@dynamic associatedObject;
	
	-(void)setAssociatedObject:(id)associatedObject {
	    objc_setAssociatedObject(self, @selector(associatedObject), associatedObject, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
	}
	
	-(id)associatedObject {
	    return objc_getAssociatedObject(self, @selector(associatedObject));
	}
	@end
	
使用场景：

1. 为现有的类添加私有变量
2. 为现有的类添加公有属性
3. 为KVO创建一个关联的观察者。

原文解释太深奥，虽然能大概懂，但是我还是不想懂。

六：动态的增加方法
已经在第一节讲过了。

七：NSCoding的自动归档和自动解档

	-(void)encodeWithCoder:(NSCoder *)aCoder {
	    unsigned int count = 0;
	    Ivar * vars = class_copyIvarList([self class], &count);
	    for (int i = 0; i < count; i++) {
	        Ivar var = vars[i];
	        const char* name = ivar_getName(var);
	        NSString * key = [NSString stringWithUTF8String:name];
	        id value = [self valueForKey:key];
	        [aCoder encodeObject:value forKey:key];
	    }
	}
	
	-(instancetype)initWithCoder:(NSCoder *)aDecoder {
	    if (self = [super init]) {
	        unsigned int count = 0;
	        Ivar * vars = class_copyIvarList([self class], &count);
	        for (int i = 0; i < count; i++) {
	            Ivar var = vars[i];
	            const char* name = ivar_getName(var);
	            NSString * key = [NSString stringWithUTF8String:name];
	            id value = [aDecoder decodeObjectForKey:key];
	            [self setValue:value forKey:key];
	        }
	    }
	    return self;
	}
	
class_copyIvarList用来获取当前model的所有成员变量。ivar_getName用来获取成员变量的名称。

八:自动模型互相转换

字典转模型

1. 调用 class_getProperty 方法获取当前 Model 的所有属性。
2. 调用 property_copyAttributeList 获取属性列表。
3. 根据属性名称生成 setter 方法。
4. 使用 objc_msgSend 调用 setter 方法为 Model 的属性赋值（或者 KVC）

		+(id)objectWithKeyValues:(NSDictionary *)aDictionary {
		    id objc = [[self alloc] init];
		    for (NSString * key in aDictionary.allKeys) {
	        id value = aDictionary[key];
        	
        //判断当前属性是不是Model
        objc_property_t property = class_getProperty(self, key.UTF8String);
        unsigned int outCount = 0;
        objc_property_attribute_t * attributeList = property_copyAttributeList(property, &outCount);
        objc_property_attribute_t attribute = attributeList[0];
        NSString * typeString = [NSString stringWithUTF8String:attribute.value];
        
        if ([typeString isEqualToString:@"@\"Diplomat\""]) {
            value = [self objectWithKeyValues:value];
        }
        
        //生成setter方法，并用objc_msgsend调用
        NSString * methodName = [NSString stringWithFormat:@"set%@%@:", [key substringToIndex:1].uppercaseString, [key substringFromIndex:1]];
        SEL setter = sel_registerName(methodName.UTF8String);
        if ([objc respondsToSelector:setter]) {
            ((void (*) (id,SEL,id)) objc_msgSend) (objc,setter,value);
        }
        free(attributeList);
    	}
    	return objc;
		}
		

模型转字典：

1. 调用 class_copyPropertyList 方法获取当前 Model 的所有属性。
2. 调用 property_getName 获取属性名称。
3. 根据属性名称生成 getter 方法。
4. 使用 objc_msgSend 调用 getter 方法获取属性值（或者 KVC）

	-(NSDictionary *)keyValuesWithObject {
	    unsigned int outCount = 0;
	    objc_property_t * propertyList = class_copyPropertyList([self class], &outCount);
	    NSMutableDictionary * dict = [NSMutableDictionary dictionary];
	    for (int i = 0; i < outCount; i++) {
	        objc_property_t property = propertyList[i];
	        
	        const char * propertyName = property_getName(property);
	        SEL getter = sel_registerName(propertyName);
	        if ([self respondsToSelector:getter]) {
	            id value = ((id (*) (id, SEL)) objc_msgSend) (self, getter);
	            
	            //判断当前属性是不是Model
	            if ([value isKindOfClass:[self class]] && value) {
	                value = [value keyValuesWithObject];
	            }
	            
	            if (value) {
	                NSString * key = [NSString stringWithUTF8String:propertyName];
	                [dict setObject:value forKey:key];
	            }
	        }
	    }
	    free(propertyList);
	    return dict;
	}
	
runtime缺点:
method swizzling不是原子性操作，+load方法里写不会有问题，但是在initialize会出现奇怪的问题。
命名冲突
	
结束语：
