---
title:collections
header: collections
description: collections
---

插个题外话

NSTaggedPointerString copy 还是原来的类和指针指向的地址没变。
NSCFConstantString copy 也是同理
NSCFString copy 还是NSCFString 但是指针指向的地址变了
stringWithFormat 创建的一定长度的字符串是TaggedPointer、否则是CFString
通过快捷方式创建的都是CFConstantString， 比如Str = @"1";

总结一下：不可变对象的copy 是浅拷贝，也就是指针copy


![???](https://Jeremy1221.github.io/img/collections_intro_2x.png)

Arrays (NSSingleObjectArray、NSArray)

Dictionarys (NSSingleEntryDictionary, NSDictionaryI)

Sets(NSSet、NSMutableSet、NSCountSet)

NSIndexSet

NSIndexPath: While they are not collections in the strictest sense, index paths ease the task of managing nested arrays. The UITableView class makes extensive use of index paths to store locations within a table view.

NSPointArray (MutableArray)

NSMapTable (MutableDictionary)

NSHashTable (MutableSet)

The thress pointer collection classes allow additional options for specifying how the collection manages its contents. You can, for example, use pointer equally instead of invoking isEqual: during comprasion. Unlike all other collection classes, NSPointerArray is allowed to hold a NULL pointer.


There are some tasks which are shared in similar form between the collection classes. Two of these tasks are copying a collection and enumeratin its contents.

In  a deep copy each object is sent a copyWithZone: message as it is added to the collection, instead of being retained.

There are two main methods of enumeration:fast enumeration and block-base enumeration.


NSDictionary 
Using custom Keys, object must conform to the NSCopying protocol. Dictionary copies each key, it is typically bad design to use large objects. Key must implement the hash and isEqual: method. performance in a dictionary is heavily dependent on the hash function used.


There are two ways to make deep copies of a collection. You can use the collection’s equivalent of initWithArray:copyItems: with YES as the second parameter. If you create a deep copy of a collection in this way, each object in the collection is sent a copyWithZone: message. If the objects in the collection have adopted the NSCopying protocol, the objects are deeply copied to the new collection, which is then the sole owner of the copied objects. If the objects do not adopt the NSCopying protocol, attempting to copy them in such a way results in a runtime error. However, copyWithZone: produces a shallow copy. This kind of copy is only capable of producing a one-level-deep copy. If you only need a one-level-deep copy, you can explicitly call for one as in Listing 2.

    NSArray *deepCopyArray=[[NSArray alloc] initWithArray:someArray copyItems:YES];

Copying and Mutability
When you copy a collection, the mutability of that collection or the objects it contains can be affected. Each method of copying has slightly different effects on the mutability of the objects in a collection of arbitrary depth:

copyWithZone: makes the surface level immutable. All deeper levels have the mutability they previously had.
initWithArray:copyItems: with NO as the second parameter gives the surface level the mutability of the class it is allocated as. All deeper levels have the mutability they previously had.
initWithArray:copyItems: with YES as the second parameter gives the surface level the mutability of the class it is allocated as. The next level is immutable, and all deeper levels have the mutability they previously had.
Archiving and unarchiving the collection leaves the mutability of all levels as it was before.


![Collection Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Collections/Articles/pointer.html#//apple_ref/doc/uid/TP40010199-SW1)
