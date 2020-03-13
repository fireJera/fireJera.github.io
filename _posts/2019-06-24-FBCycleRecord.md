---
title: FBStudyRecord
header: FBStudyRecord
description: FBStudyRecord
---

# Love is touch and yet not a touch --Jerome David Salinger

1. namespace 命名空间 可以嵌套并且不连续
2. virtual 虚函数 子类重写 并且当前类调用当前的方法
3. ~析构函数 当我们创建一个有参构造函数时 编译器不会创建默认的构造函数。我们自己生命默认构造函数并在末尾加上=default来告诉编译器去实现这个方法 A()=default  不比写成这样 A(){} = default;
4. =delete c++11之前释放动态分配内存 c++11之后 禁用成员函数的使用 （即是将此方法给禁用）
5. 构造函数  
    class Unresolved: public BaseType {
    public:
        std::string value;
        Unresolved(std::string value): value(value) {}
    }

最后一行的构造函数 应该是将方法传入的参数value 复制给 属性value

6. using 1.命名空间 2.使用父类的成员 3. 别名指定
7. auto 自动获取类型
8. 

# 各个类的作用
1. FBReatinCycleDetector
 // FBObjectGraphConfiguration
- (instantype)init;

- (void)initWithConfiguration:(FBObjectGraphConfiguration *)configuration 初始化方法

// 传入你想监听循环引用的对象 内部使用NSMutableArray
// 将参数 转换为 FBObjectiveCGraphElement 对象 加入数组
// 这里的转换使用了 FBRetainCycleUtils 中的C方法来生成FBObjectiveCGraphElement对象 
- (void)addCandidate:(id)candidate; 

// 在所有备选者中查找循环引用， 最大长度为10
- (nonnull NSSet<NSArray<FBObjectiveCGraphElement *> *> *)findRetainCycles;

// 在所有备选者中查找循环引用， 超过10时使用此方法
- (nonnull NSSet<NSArray<FBObjectiveCGraphElement *> *> *)findRetainCyclesWithMaxCycleLength:(NSUInteger)length;

// 单元测试
- (NSArray *)_shiftToUnifiedCycle:(NSArray *)array;

1.1 FBRetainCycleUtils 中定义了两个C方法-创建FBObjectiveCGraphElement 

// 内部实现 : 1.  通过内部方法 _ShouldBreakGraphEdge判断是否需要继续转换 newElement
// 如果传入对象为Block newElement = FBObjectiveCBlock
// 如果为NSTimer  newElement = FBObjectiveCNSCFTimer
// 否则 newElement = FBObjectiveCObject
// 如果 configuration 有transfromerBlock 调用tranformerBlock 否则返回 newElement
FBObjectiveCGraphElement *_Nullable FBWrapObjectGraphElementWithContext(FBObjectiveCGraphElement *_Nullable sourceElement,
                                                                        id _Nullable object,
                                                                        FBObjectGraphConfiguration *_Nullable configuration,
                                                                        NSArray<NSString *> *_Nullable namePath);
                                                                        
FBObjectiveCGraphElement *_Nullable FBWrapObjectGraphElement(FBObjectiveCGraphElement *_Nullable sourceElement,
                                                             id _Nullable object,
                                                             FBObjectGraphConfiguration *_Nullable configuration);

// .m 内部方法
// 根据传入的configuration的filterblocks 遍历执行 如果 返回结果 == FBGraphEdgeInvalid return YES
// 全部遍历完返回NO
static BOOL _ShouldBreakGraphEdge(FBObjectGraphConfiguration *configuration,
                                  FBObjectiveCGraphElement * fromObject,
                                  NSString *byIvar,
                                  Class toObjectOfClass);

2. FBObjectGraphConfiguration
头文件中有一个枚举 FBGraphEdgeType 图表边界类型？ 有两个值 FBGraphEdgeTypeValid FBGraphEdgeTypeInvalid
两个全局 block 

根据传入的detector中addCandidate传入的参数转换后的对象、 返回这个对象的是否可以根据内部布局去判断有强引用
typedef FBGraphEdgeType (^FBGraphEdgeFilterBlock)(FBObjectiveCGraphElement *_Nullable fromObject,
                                                  NSString *_Nullable byIvar,
                                                  Class _Nullable toObjectOfClass);

转换block 将一个FBObjectiveCGraphElement 对象 转换成另一个?
typedef FBObjectiveCGraphElement *_Nullable(^FBObjectiveCGraphElementTransformerBlock)(FBObjectiveCGraphElement *_Nonnull fromObject);

下面是这个类的属性和方法

@property NSArray<FBGraphEdgeFilterBlock> *filterBlocks
@property FBObjectiveCGraphElementTransformerBlock transformerBlock
@property BOOL shouldInspectTimers
@property BOOL shouldIncludeBlockAddress
@property NSMutableDictionary<Class, NSArray<id<FBObjectReference>> *> *layoutCache;
@property BOOL shouldCacheLayouts

- (nonnull instancetype)initWithFilterBlocks:(nonnull NSArray<FBGraphEdgeFilterBlock> *)filterBlocks
                         shouldInspectTimers:(BOOL)shouldInspectTimers
                         transformerBlock:(nullable FBObjectiveCGraphElementTransformerBlock)transformerBlock
                         shouldIncludeBlockAddress:(BOOL)shouldIncludeBlockAddress NS_DESIGNATED_INITIALIZER;

- (nonnull instancetype)initWithFilterBlocks:(nonnull NSArray<FBGraphEdgeFilterBlock> *)filterBlocks
                         shouldInspectTimers:(BOOL)shouldInspectTimers
                         transformerBlock:(nullable FBObjectiveCGraphElementTransformerBlock)transformerBlock;

- (nonnull instancetype)initWithFilterBlocks:(nonnull NSArray<FBGraphEdgeFilterBlock> *)filterBlocks
                         shouldInspectTimers:(BOOL)shouldInspectTimers;

// 默认filterBlocks 为空 shouldInperctTimers 为yes
- (instancetype)init;

3. FBObjectiveCGraphElement

// 在RETAIN_CYCLE_DETECTOR_ENABLED 下 
// 获取object的类 allowsWeakReference = 获取这个类的allowsWeakReference的MethodImplementation
// if allowsWeakReferenct && (IMP)allowsWeakReferenct != _objc_msgForward  
// if (allowsWeakReference(object, @selector(allowsWeakReference))) _object = object
// else _object = object
// https://www.jianshu.com/p/eff6b9443800
- (nonnull instancetype)initWithObject:(nullable id)object 
                         configuration:(nonnull FBObjectGraphConfiguration *)configuration
                              namePath:(nullable NSArray<NSString *> *)namePath;

- (nonnull instancetype)initWithObject:(nullable id)object
                         configuration:(nonnull FBObjectGraphConfiguration *)configuration;

- (instancetype)initWithObject:(id)object;

@property (nonatomic, copy, readonly, nullable) NSArray<NSString *> *namePath;
@property (nonatomic, weak, nullable) id object;
@property (nonatomic, readonly) FBObjectGraphConfiguration * configuration;

// 返回通过runtime objb_setAssociatedObject 强引用的对象
- (nullable NSSet *)allRetainObjects;

// (size_t)_object
- (size_t)objectAddress;
// object_getClass(_object)
- (nullable Class)objectClass;
// NSStringFromClass()
- (NSString *)classNameOrNull;
// 判断引用的object 是否相等
- (BOOL)isEqual:(id)object;
// 返回(size_t) 应该是指针
- (NSUInteger)hash;
4. FBObjectiveCBlock

内部有一个结构体  
struct __attribute__((packed)) BlockLiteral {
    void *isa;
    int flags;
    int reserved;
    void *invoke;
    void *descriptor;
};

// 先调用父类方法 获取关联强引用
// 然后通过FBBlockStrongLayout中的C方法FBGetStrongReferences获取强引用 将结果数组转成NSSet
- (nullable NSSet *)allRetainObjects;

// className = NSStringFromClass 如果为空 返回 @"(null)"
// __attribute__((objc_precise_lifetime)) id anObject = self.object; 防止过早释放 也就是在局部变量生效的开始如果有值，那么在这个局部的范围内，即时arc检测到引用计数为0该释放了，那么也不会立刻释放，这个范围的代码全部结束时释放。
// 如果self.object isKindOfClass FBBlockStrongRelationDetector class anObject = [ forwarding]
// 将anObject转为指针 为空返回 className
// 转为BlockLiteral 指针 然后获取其调用函数 invoke
// 打印 className 和 invoke方法的地址
//
- (NSString *)classNameOrNull

5. FBObjectiveCNSCFTimer
继承于FBObjectiveCObject

内部结构体 _FBNSCFTimerInfoStruct {
    long _unknow;
    id target;
    SEL selector;
    NSDictionary * userInfo;
}

/*
又用到了 这个东西__attribute__((objc_precise_lifetime)) 一句话解释保证不会提前释放
timer = self.object 引用NSTimer 调用父类的allRetainObjects 可变集合retained接收
使用了runloop的一些方法 CFRunLoopTimerGetContext 来获取timer的context
如果 context的info 和 retain 同时为真 将context.info 转为上述结构体
如果结构体的target非空 把target 转换为Element的子类 retained加入element
如果userInfo非空 同上 最后返回 retained
*/
- (NSSet *)allRetainObjects;

6. FBObjectiveCObject

/*
1. call FBGetObjectStrongReferences(object, layoutCache)
调用父类的allReatinObjects 可变数组接收 遍历返回的strong ivars 
根据自己的object和便利的ivarReference获取对应的实际值 再获取ivar的变量名
调用 FBWrapObjectGraphElementWithContext 将Ivar 装换成 FBObjectiveCGraphElement的子类
加入到上面的可变数组中  如果自己的object的class 有__NSCF前缀返回 可变数组的NSSet 对象
如果aCls 是元类 返回nil 如果这个类遵守了NSFastEnumeration  
调用 _objectRetainsEnumerableKeys 判断是否有强引用Keys  _objectRetainsEnumerableKeys 判断是否有强引用values
判断aCls 是否可以通过objectForKey: 来取值

接下来 为了防止在枚举过程中出错 会设定一个重试次数 枚举失败则继续 直到成功会次数达到
使用forin  声明 mutableselt 枚举self.object 如果有强引用key 枚举的对象则是key FBWrapObjectGraphElement 生成新的 element 子类对象
如果同时含有强引用key和value 通过objectForKey 步骤同上 获得element子类对象 加入到 mutableselt
最后 加入可变数组中 然后返回可变数组
*/
- (NSSet *)allRetainObjects
// 判断是否含有强引用的values
- (BOOL)_objectReatinsEnumerableValues
// 判断是否含有强引用的keys
- (BOOL)_objectRetainsEnumerableKeys

7. FBBlockStrongRelationDetector
{
    // block fakery
    void *forwarding;
    int flags;      //refCount
    int size;
    void(*byref_keep)(struct _block_byref_block *dst, struct _block_byref_block *src);
    void (*byref_dispose)(struct _block_byref_block *);
    void *captured[16];
}

@property (nonatomic, assign, getter=isStrong) BOOL strong;
// 调用[super release]
- (oneway void)trueRelease;
// __block引用的变量
- (void *)forwarding;

static void byref_keep_nop(struct _block_byref_block *dst, struct _block_byref_block *src){}
static void byref_dispose_nop(struct _block_byref_block *param) {}


// 如果调用了此方法 ——strong = yes
release
// forwarding  = obj
// byref_keep = byref_keep_nop
// byref_dispose =byref_dispose_nop
// return obj
alloc


/**
 Name path that describes how this object was retrieved from its parent object by names
 (for example ivar names, struct references). For more check FBObjectReference protocol.
 */
@property (nonatomic, copy, readonly, nullable) NSArray<NSString *> *namePath;
@property (nonatomic, weak, nullable) id object;
@property (nonatomic, readonly, nonnull) FBObjectGraphConfiguration *configuration;

/**
 Main accessor to all objects that the given object is retaining. Thread unsafe.

 @return NSSet of all objects this object is retaining.
 */
- (nullable NSSet *)allRetainedObjects;

/**
 @return address of the object represented by this element
 */
- (size_t)objectAddress;

- (nullable Class)objectClass;
- (nonnull NSString *)classNameOrNull;

8. FBAssociationManager
//
+ (void)hook;
//
+ (void)unhook;
//
+ (nullable NSArray *)associationsForObject:(nullable id)object;

命名空间FB::AssociationManager里四个方法

    // 
    void _threadUnsafeResetAssociationAtKey(id object, void * key);
    //
    void _threadUnsafeSetStrongAssociation(id object, void *key, id value);
    //
    void _threadUnsafeRemoveAssociations(id object);
    //        
    NSArray * associations(id object);

9. FBIvarReference
// 
typedef NS_ENUM(NSUInteger, FBType) {
    FBObjectType,       // @
    FBBlockType,        // @?
    FBStructType,       // {
    FBUnknowType,
};

@property (nonatomic, copy, readonly, nullable) NSString * name;
@property (nonatomic, readonly) FBType type;
// ivar_getOffset
@property (nonatomic, readonly) ptrdiff_t offset; // ptrdiff_t 通常保存两个指针减操作的结果
// offset / sizeof(void *)
@property (nonatomic, readonly) NSUInteger index;
@property (nonatomic, readonly) Ivar ivar;

- (instancetype)initWithIvar:(Ivar)ivar;

10. FBNodeEnumerator : NSEnumerator

    // 初始化方法
    - (instancetype)initWithObject:(FBObjectiveCGraphElement *)object;
    /*
    如果 _object为空 返回nil else if snapshot 为空  snapshot = _object allRetainedObjects 
    _enumerator = snapshot objectEnumerator 
    element的对象next = enumerator的nextObject 如果不为空 使用next 创建新的 NodeEnumerator对象 返回
    */
    - (nullable FBNodeEnumerator *)nextObject;
    //
    @property (nonatomic, strong, readonly) FBObjectiveCGraphElement * object;

内部两个实例变量 
    
    NSSet * _retainedObjectSnapshot;
    NSEnumerator * _enumerator;

    //实现了isEqual:(id)object 方法 和hash 方法 hash 实现 返回object的hash

11. FBObjectInStructReference
实现了FBObjectReference 协议

    // 初始化方法
    - (instancetype)initWithIndex:(NSUInteger)index
                     namePath:(nullable NSArray<NSString *> *)namePath;

    // 以下是协议方法
    - (NSString *)description {
        // %td %12ti 便是ptrdiff_t
        return [NSString stringWithFormat:@"[in_struct; index: %td]", _index];
    }

    - (id)objectReferenceFromObject:(id)object {
        return FBExtractObjectByOffset(object, _index);
    }

    - (NSUInteger)indexInIvarLayout {
        return _index;
    }

    - (NSArray<NSString *> *)namePath {
        return _namePath;
    }


##C++类

1. BaseType FB::RetainCycleDetector::Parser

    class BaseType {
    public:
        // 虚函数 希望子类去重载 不要调用父类的实现
        virtual ~BaseType() {}
    };
    
    class Unresolved: public BaseType {
    public:
        std::string value;
        Unresolved(std::string value): value(value) {}
        // = default 编译器生成默认实现
        // 该函数比用户自己定义的默认构造函数获得更高的代码效率
        Unresolved(Unresolved&&) = default;
        Unresolved &operator=(Unresolved&&) = default;
        
        // Unresolved a, b  = delete 应该类似于ns_unavaiable
        // a = Unresolved(b) error
        Unresolved(const Unresolved&) = delete;
        // a = b error
        Unresolved &operator=(const Unresolved&) = delete;
    };

2. namespace FB { namespace RetainCycleDetector { namespace Parser {
    class Type: public BaseType {
    public:
        const std::string name;
        const std::string typeEncoding;
        
        Type(const std::string &name,
             const std::string &typeEncoding): name(name), typeEncoding(typeEncoding) {}
        Type(Type&&) = default;
        Type &operator=(Type&&) = default;
        
        Type(const Type&) = delete;
        Type &operator=(const Type&) = delete;
        
        virtual void passTypePath(std::vector<std::string> typePath) {
            this->typePath = typePath;
        }
        
        std::vector<std::string> typePath;
    };
}}}

3. Struct : Type 命名空间 FB::RetainCycleDetector::Parser

    const std::string structTypeName;
        
    Struct(const std::string &name,
               const std::string &typeEncoding,
               const std::string &structTypeName,
               std::vector<std::shared_ptr<Type>> &typesContainedInStruct)
    : Type(name, typeEncoding),
    structTypeName(structTypeName),
    typesContainedInStruct(std::move(typesContainedInStruct)) {};
    Struct(Struct&&) = default;
    Struct &operator=(Struct&&) = default;
        
    Struct(const Struct &) = delete;
    Struct &operator=(const Struct&) = delete;
    
    /*
    遍历typesContainedInStruct 强制类型转换成Struct 
    如果转换成了Struct
    reserve 改变capacity 当前 vector的大小 + typesContainedInStruct。size
    把整个typesContainedInStruct 插入 flatten 的尾部    
    否则直接插入
    */
    std::vector<std::shared_ptr<Type>> flattenTypes();
    
    /*
     当前typePath = typePath 然后加入自己的name 然后加上自己的structTypeName
     遍历 typesContainedInStruct 递归调用passTypePath
    */
    virtual void passTypePath(std::vector<std::string> typePath);
    std::vector<std::shared_ptr<Type>> typesContainedInStruct;

##protocol


1. FBObjectReference
// 
- (NSUInteger)indexInIvarLayout;
// 给定一个对象 获取这个对象相应的ivar的实际值
- (id)objectreferenceFromObject:(id)object);
// 引用路径 从谁开始
- (MSArray<NSString *> *)namePath;

##文件 ：
1. FBBlockStrongLayout

两个C方法 
// 先 FBObjectIsBlock 判断是否是block
// 再 _getBlockStrongLayout 返回IndexSet
// 遍历IndexSet 判断指针以及指针指向的内容是否为空 不为空 添加到数组中 返回数组
NSArray * FBGetBlockStrongReferences(void *block);
// 调用 _BlockClass 返回blockClass
// 然后 candidate = object_getclass)(object)
// 判断 candidate isSubClassOf blockClass
BOOL FBObjectIsBlock(void *)block;

内部方法 
// 将block 转换为 BlockLiteral 结构体
// 如果block 有C++构造器  或者 block 没有释放函数 忽略 如果有C++那么就没有指针对齐 
// 所以无法获取layout 如果没有释放函数 那么就不知道谁会被释放 都没法获取强引用
// 获取释放函数的引用 然后计算引用的个数
// 遍历引用的个数 生成FBBlockStrongRealtionDetector 分别加入两个数组中
// 调用释放函数 释放一个数组 然后再另一个数组中看谁没调用dealloc方法 即为强引用
static NSIndexSet * _GetBlockStrongLayout(void *)block

// dispatch_once
// 生成测试Block 然后获取block的class
// 遍历 blockClass 的 superclass 如果superclass 不为空 并且 不是NSObject class
// 这里在测试的时候发现，__NSGlobalBlock__的父类是 __NSGlobalBlock 再父类是NSBlock，NSBlock的父类是NSObject
// __NSMallocBlock__的父类是 __NSMallocBlock 再父类是NSBlock，NSBlock的父类是NSObject
// 返回blockClass
static Class _BlockClass 

2. fishhook

一个结构体

    struct rcd_rebinding {
        const char *name;
        void *replacement;
        void **replaced;
    }

    /*
    先调用rcd_prepend_rebindings  将出入的rebindings 放入链表的头 
    如果链表的头没有下一个 _dyld_register_func_for_add_image(_rebind_symbols_for_image);
    设定动态库加载时的回调
    获取 imagecount 遍历 调用
    _rebind_symbols_for_image(_dyld_get_image_header(i), _dyld_get_image_vmaddr_slide(i));
    一个拿到image的section_header 一个拿到 image的虚拟内存偏移地址
    */
    int rcd_rebind_symbols(struct rcd_rebinding rebindings[], size_t rebinds_net);
    /*
    
    */
    int rcd_rebind_symbols_image(void *header, intptr_t slide, struct rcd_binding rebindings[], size_t rebindings_nel);

    内部方法

    内部结构体

    struct rcd_rebindings_entry {
        struct rcd_rebinding *rebindings;
        size_t rebindbings_nel; 
        struct rcd_rebindings_entry *next;
    };

    //静态结构体指针 链表的头
    static struct rcd_rebindings_entry * _rebindbings_head;
    // 实际是调用  rebind_symbols_for_image
    static void _rebind_symbols_for_image(const struct mach_header *header, intptr_t slide) 
    /*
    1 dladdr() 从头header中获取地址的符号信息
    2 header的起始地址+header的大小 = header的结束地址 = header内容的起始地址
    3 遍历header中cmd的个数 分别拿到 linkedit_segment symtab_cmd dysymtab_cmd
    4 算出初始地址 然后根据symtab_cmd 的stroff 和 symoff 得到符号表 和 字符串表 以及间接符号表
    5 遍历header中cmd的个数 如果segment是__DATA__ 跳过 然后在segment中遍历section的个数
    如果是 S_LAZY_SYMBOL_POINTERS or S_NON_LAZY_SYMBOL_POINTERS rcd_perform_rebinding_with_section
    */
    static void rebind_symbols_for_image(struct rcd_rebindings_entry * rebindings,
                                     const struct mach_header *header,
                                     intptr_t slide)
    /*
    1 根据间接符号表和setction的偏移量 或下标 
    2 
    3 
    4 
    5 
    */
    static void rcd_perform_rebinding_with_section(struct rcd_rebindings_entry *rebindings,
                                               section_t *section,
                                               intptr_t slide,
                                               nlist_t *symtab,
                                               char *strtab,
                                               uint32_t *indirect_symtab)


    // 先malloc初始化一个 new_entry = rcd_rebindings_entry 结构体 初始化失败返回-1
    // malloc初始化rebindings 属性的结构体 失败 返回-1
    // 把参数 rebindings[] memcpy到 new_entry->rebindings new_entry->next = head head = new_entry
    // 就是一个单向链表 
    // 总的来说就是 将rebindings_head 存入rcd_rebindings_entry 插入头部
    static int rcd_prepend_rebindings(struct rcd_rebindings_entry **rebindings_head,
                                     struct rcd_rebinding rebindings[],
                                     size_t nel)

3. FBClassStrongLayout 

两个C方法
/*
传入 Class 
获取ivarList 遍历生成FBIvarReference 如果ivar 是struct call FBGetReferencesForObjectsInStructEncoding  加返回的数组 加入数组result 中
else result 直接加入 FBIvarReference  return result
*/
NSArray<id<FBObjectReference>> * FBGetClassReferences(__unsafe_unretained Class _Nullable aCls);
    
/*
遍历 superclass 直到当前的class != superclass
先从 缓存中 获取obj的class的强引用内存布局 
如果没有找到 FBGetStrongReferencesForClass 获取布局 将布局存入参数layoutCache中 放入 array中 
return array
 */
NSArray<id<FBObjectReference>> * FBGetObjectStrongReferences(id _Nullable obj,
                                                            NSMutableDictionary<Class, NSArray<id<FBObjectReference>> *> *_Nullable layoutCache);

内部方法
/*
传入两个参数 一个 FBIvarReference ivar  一个std::string encoding
ivarName = ivar.name 调用 parseStructEncodingWithName(ivarName, encoding) 在获取返回的Struct的flattenTypes
offset = ivar.offset 遍历 types 如果type->encoding [0] = '^' 指针 size = sizeof(void *) align = _Align(void *)
else NSGetSizeAndAlignment 
overAlign = offset % align  whatMissing = overAlign == 0 ? 0 : align - overAlign offset+=whatMissing
如果 encoding 是对象 遍历type的typePath 加入namePath中 如果type you name 也加入

最后 生成 新的FBObjectInStructReference 对象 加入 refereneces中 
offset += size
返回references
*/
static NSArray *FBGetReferencesForObjectsInStructEncoding(FBIvarReference *ivar, std::string encoding)
/*
minimumIndex const uint8_t * layoutDescription
根据传入的strong layout 遍历 获取 strong ivar 在类中的分布 标签
*/
static NSIndexSet *FBGetLayoutAsIndexesForDescription(NSUInteger minimumIndex, const uint8_t *layoutDescription)
/*
获取第一个ivar 在布局中的index
*/
static NSUInteger FBGetMinimumIvarIndex(__unsafe_unretained Class aCls)
/*
Class aCls 
根据传入的aCls call FBGetClassReferences 然后 过滤掉 未知类型的ivar 
获取aCls的layout FBGetminimumIvarIndex 然后 FBGetLayoutAsIndexesForDescription 获取 强引用ivar的下表
在类的所有确定类型ivar 中遍历 过滤掉弱引用的ivar
*/
static NSArray<id<FBObjectReference>> *FBGetStrongReferencesForClass(Class aCls)

4 FBStructEncodingParser

命名空间FB::RetainCycleDetector::Parser

    Struct Struct parserStructEncodig(const std::string &structEncodingString);
    
    Struct parseStructEncodingWithName(const std::string &structEncodingString,
                                        const std::string &structName);

    内部类 _StringScanner 

    class _StringScanner {
    public:
        const std::string string;
        size_t index;
        
        _StringScanner(const std::string &string): string(string), index(0) {}
        
        /*
        给定一个字符串 然后在index位开始比较 不相等返回false 相等 index 向后移动string的length
        */
        bool scanString(const std::string &stringToScan) {
            if (!(string.compare(index, stringToScan.length(), stringToScan) == 0)) {
                return false;
            }
            index += stringToScan.length();
            return true;
        }
        /*
        从index 位开始查找upToString 如果没找到 返回"" index = length
        找到后 返回 index到结果中间的字符串 index += length
        */
        const std::string scanUpToString(const std::string &upToString) {
            size_t pos = string.find(upToString, index);
            if (pos == std::string::npos) {
                // Mark as whole string scanned
                index = string.length();
                return "";
            }
            std::string inBetweenString = string.substr(index, pos - index);
            index = pos;
            return inBetweenString;
        }
        
        const char currentCharacter() {
            return string[index];
        }
        
        /*
        找到第一个字符串
        */
        const std::string scanUpToCharacterFromSet(const std::string &charactreSet) {
            size_t pos = string.find_first_of(charactreSet, index);
            if (pos == std::string::npos) {
                index = string.length();
                return "";
            }
            
            std::string inBetweenString = string.substr(index, pos - index);
            index = pos;
            return inBetweenString;
        }
    };

内部 struct _StructParseResult

struct _StructParseResult {
        std::vector<std::shared_ptr<Type>> containedTypes;
        const std::string typeName;
    };
    
    static const auto kOpenStruct = "{";
    static const auto kCloseStruct = "}";
    static const auto kLiteralEndingCharacters = "\"}";
    static const auto kQuote = "\"";
    
    /*
    vector<BaseType> tyoes
    先在scanner的string 中查找{ 如果没找到 NSCAssert
    然后在查找'=' 获取 index位到=号中间的字符串 再次查找"="
    遍历查找 } 找到就退出循环
    遍历过程中如果找到 \" 获取 当前位到 结果位的字符串 查找下一个 \" 如果 将获取的字符串 转成Unresolved加入types
    如果scanner的标签位 为{ 字符递归调用本方法 result = 返回的 _StructParseResult 
        如果 types 不是空的 拿到types的最后一个转成Unresolved 然后删除 types的最后一个 如果转成了Unresolved 在根据 Unresolved的value 
        生成新的Struct 加入到types中
    else 找到第一个 \"} 保存结果 result 如果types 不为空 将types的最后一个转换为Unresolved nameBefore = 最后一个的 value 删除最后一个
    生成新的Type(nameBefore, result)  加入到types中

    遍历types 将每一个成员强制装换 Type 加入新的 filteredVector return {filteredVector, structTypeName};
    */
    static struct _StructParseResult _ParseStructEncodingWithScanner(_StringScanner &scanner,
                                                                     NSString *debugStruct) {
        std::vector<std::shared_ptr<BaseType>> types;
        
        // Every struct starts with '{'
        __unused const auto scannedCorrectly = scanner.scanString(kOpenStruct);
        NSCAssert(scannedCorrectly, @"The first characte of struct encoding should be {;debug_struct:%@", debugStruct);
        
        // Parse name
        const auto structTypeName = scanner.scanUpToString("=");
        scanner.scanString("=");
        
        while (!(scanner.scanString(kCloseStruct))) {
            if (scanner.scanString(kQuote)) {
                const auto parseResult = scanner.scanUpToString(kQuote);
                scanner.scanString(kQuote);
                if (parseResult.length() > 0) {
                    types.push_back(std::make_shared<Unresolved>(parseResult));
                }
            } else if (scanner.currentCharacter() == '{') {
                // We do not want to consume '{' because we will call parser recursively
                const auto locBefore = scanner.index;
                auto parseResult = _ParseStructEncodingWithScanner(scanner, debugStruct);
                
                std::shared_ptr<Unresolved> valueFromBefore;
                if (!types.empty()) {
                    valueFromBefore = std::dynamic_pointer_cast<Unresolved>(types.back());
                    types.pop_back();
                }
                const auto extractedNameFromBefore = valueFromBefore ? valueFromBefore->value : "";
                
                std::shared_ptr<Struct> type = std::make_shared<Struct>(extractedNameFromBefore,
                                                                        scanner.string.substr(locBefore,
                                                                                              (scanner.index - locBefore)),
                                                                        parseResult.typeName,
                                                                        parseResult.containedTypes);
                
                types.emplace_back(type);
            } else {
                // It's a type name (literal), let's advance until we find '"', or '}'
                const auto parseResult = scanner.scanUpToCharacterFromSet(kLiteralEndingCharacters);
                std::string nameFromBefore = "";
                if (types.size() > 0) {
                    if (std::shared_ptr<Unresolved> maybeUnresolved = std::dynamic_pointer_cast<Unresolved>(types.back())) {
                        nameFromBefore = maybeUnresolved->value;
                        types.pop_back();
                    }
                }
                std::shared_ptr<Type> type = std::make_shared<Type>(nameFromBefore, parseResult);
                types.emplace_back(type);
            }
        }
        
        std::vector<std::shared_ptr<Type>> filteredVector;
        
        for (const auto &t: types) {
            // dynamic_pointer_cast 强制类型转换
            if (const auto convertedType = std::dynamic_pointer_cast<Type>(t)) {
                filteredVector.emplace_back(convertedType);
            }
        }
        return {
            .containedTypes = filteredVector,
            .typeName = structTypeName,
            };
        }
        /*
        调用 parseStructEncodingWithName 传入 空structName
        */
        Struct parseStructEncoding(const std::string &structEncodingString) {
            return parseStructEncodingWithName(structEncodingString, "");
        }
        /*
        创建结构体 根据encoding 调用 _PareseStructEncodingWithScanner()
        返回的resuklt的typeName和containedTypes 创建Struct
        调用passTypePath传入空字符串 返回
        */
        Struct parseStructEncodingWithName(const std::string &structEncodingString,
                                           const std::string &structName) {
            _StringScanner scanner = _StringScanner(structEncodingString);
            auto result = _ParseStructEncodingWithScanner(scanner,
                                                          [NSString stringWithCString:structEncodingString.c_str()
                                                                             encoding:NSUTF8StringEncoding]);
            
            Struct outerStruct = Struct(structName,
                                        structEncodingString,
                                        result.typeName,
                                        result.containedTypes);
            outerStruct.passTypePath({});
            return outerStruct;
        }


5. FBBlockInterface

We are mimcing Block structure based on Clang documentation:
http://clang.llvm.org/docs/Block-ABI-Apple.html

一个枚举 两个结构体

    enum {
        BLOCK_IS_NONESCAPE          = (1 << 23)

        BLOCK_HAS_COPY_DISPOSE      = (1 << 23)
        BLOCK_HAS_CTOR              = (1 << 23)
        BLOCK_IS_GLOBAL             = (1 << 23)
        BLOCK_HAS_STRET             = (1 << 23)
        BLOCK_HAS_SIGNATURE         = (1 << 23)
    }   

    struct BlockDescriptor {
        unsigned long reserved;
        unsigned long size;

        void(* copy_helper)(void *dst, void *src);      //IFF (1 << 25)
        void(* dispose_helper)(void *src);              //IFF (1 << 25)
        const char * signature;                         //IFF (1 << 30)
    }

    struct BlockLiteral {
        void isa;
        int flags;
        int reserved;
        void(* invoke)(void *, ....);
        struct BlockDescriptor * descriptor;
        // imported variables
    }

6. FBClassStrongLayoutHelpers
 一个C方法
    
    /*
    给定一个对象 给定一个索引 返回该索引 的对象值
    将obj 转成指针 然后 + index * sizeof(void *)
    */
    id FBExtractObjectByOffset(id obj, NSUInteger index);

7. FBStandardGraphEdgeFilters
4个C方法

    /*
    holdActionClass = NSClassFromString(@"UIHeldAction")
    transitionContextClass = @"_UIViewControllerOneToOneTransitionContext"
    */
    NSArray<FBGraphEdgeFilterBlock> *FBGetStandardGraphEdgeFilters(void) {
        return @[FBFilterBlockWithObjectIvarRelation([UIView class], @"_subviewCache"),
             FBFilterBlockWithObjectIvarRelation(heldActionClass, @"m_target"),
             FBFilterBlockWithObjectToManyIvarsRelation([UITouch class], [NSSet setWithArray:@[@"_view",
                                                                                               @"_gestureRecognizers",
                                                                                               @"_window",
                                                                                               @"_warpedIntoView"]]),
             FBFilterBlockWithObjectToManyIvarsRelation(transitionContextClass,
                                                        [NSSet setWithArray:@[@"_toViewController",
                                                                              @"_fromViewcontroller"]])];                                                                

                                                                    }

    // call  FBFilterBlockWithObjectToManyIvarsRelation                                                                
    FBGraphEdgeFilterBlock FBFilterBlockWithObjectIvarRelation(Class aCls,
                                                            NSString *ivarName);

    /*
    如果element的object是给定的类的的子类 并且给定的names集合里有ivar的 忽略掉

    返回一个 filterBlock 这个block 传入三个参数 fromObject- element 的子类 byIvar 变量的名称 toObjectClass-element的object的ivar的值的class
    判断[fromObject objectClass] 是否是aCls的子类 如果ivarNames 包含byivar 返回不可用 否则返回可用
    */                                                         
    FBGraphEdgeFilterBlock FBFilterBlockWithObjectToManyIvarsRelation(Class aCls,
                                                                    NSSet<NSString *> *ivarNames);

    /*
    如果ivar的class  是给定的toclass的子类 再调用上述方法
    返回一个 filterBlock 这个block 传入三个参数 fromObject- element 的子类 byIvar 变量的名称 toObjectClass-element的object的ivar的值的class
    如果toClass非空 并且 ivar的class 是 toClass的子类 返回 FBFilterBlockWithObjectIvarRelation  最后返回可用
    */   
    FBGraphEdgeFilterBlock FBFilterBlokWithObjectIvarObjectRelation(Class fromClass,
                                                                    NSString *ivarName,
                                                                    Class toClass) 

                                                                    

# 工作流
1. FBRetainCycleUtils初始化一个实例，然后实例调用addCandidate:加入一个要检测的实例 放入一个数组中，放入数组前转换为相应的Timer、Block或者Object类的实例，
2. 然后调用findRetainCycles方法去遍历数组中的对象，在遍历每一个对象时，首先将每一个对象转换为FBNodeEnumerator（一个能够遍历的类似于NSArray的自定义容器）。
3. 通过每个对象的allRetainObjects，方法里通过[FBAssiciationManager associationsForObject:对象]获取到runtime关联的引用数组,FBAssiciationManager有个hook和unhook方法通过fishhook.h在加载动态库时将指定方法1与方法t2替换，类似于method_swiizzling，在每次设置值进去的时候，会自动调用交换的方法，然后获取到给定类的强引用的数组。
4.  再回到上一步，a根据a的不同类型调用不同类的allRetainObjects方法，在a的父类的allRetainObjects方法里上面第一步已经获取到了关联的强引用的值，将每一个值转换为FBObjectiveCGraphElement 放入到一个新的数组中，返回数组。
4.2 然后block类的allRetainObjects原理 : 调用父类也就是关联的属性，然后将block转换为BlockLiteral结构体指针。如果结构体有C++构造器或者没有copy、dispose方法，返回空。通过将block转换成struct，然后根据结构体中指针的数量生成对应数量的对象FBBlockStrongLayout，调用dispose_helper然后就能知道对象哪个位置的变量是strong和weak，再获取对应位置的指针转换为具体对象（取地址&和下标），再去遍历每个对象有没有强引用其他的对象。
4.3 timer allRetainObjects 原理：NSTimer.target、userinfo、关联属性、强引用、父类的强引用（知道父类和父类的父类相等）、将NSTimer通过runloop获取上下文然后转换成_FBNSCFTimerInfoStruct结构体
4.4 object allRetainObjects原理：关联属性、强引用、父类的强引用（知道父类和父类的父类相等）通过IvarLayout 来 识别什么是哪些是strong、哪些是weak
4.5 如果ivar里有结构体，然后会扫描ivar的encode 获取结构体名字。以及结构体的各个变量及类型。
5. 回到第三步开始，递归遍历对象的所有强关联属性，属性的强关联属性，如果找到放到第一位。
6
7
8
9
10
# 



###结构体字节对齐
首先几个概念：
自身对齐值：成员中最大的一个对齐值。
指定对齐值：#pragma pack(value) value指定的值 value 只能填1、2、4、8、16。
有效对齐值：上述两个最小的那个。

步骤首先确定cpu周期(看系统或编译器是多少位对齐的，一般除了4位就是8位了)、自身对齐值、指定对齐值。实际对齐值就是上面最小的。
