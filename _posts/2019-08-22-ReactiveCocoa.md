---
title: ReactiveCocoa
header: ReactiveCocoa
description: ReactiveCocoa
---

# When your mind's made up, there's no point trying to change it.

## Folder - extobjc

1. EXTKeyPathCoding.h

    @keypath  
    //https://www.jianshu.com/p/11427003fa9b


2. EXTRuntimeExtension.h
    // 枚举
    rac_propertyMemoryManagementPolicyAssign = 0,
    rac_propertyMemoryManagementPolicyRetain,
    rac_propertyMemoryManagementPolicyCopy

    // 结构体
    typedef struct {    
        BOOL readonly;
        BOOL nonatomic;
        BOOL weak;
        BOOL canBeCollected;
        BOOL dynamic;
        rac_propertyMemoryManagementPolicy memoryManagementPolicy;
        SEL getter;
        SEL setter;
        const char *ivar;
        Class objectClass;
        char type[];
    } rac_propertyAttributes;

    // 获取实例方法 从methodList中遍历查找  如果找到返回 否则返回NULL
    Method rac_getImmediateInstanceMethod (Class aClass, SEL aSelector);
    // 获取属性的结构体
    /*
    1. 如果拿不到attributes 返回 NULL
    2. 如果第一个字母不是T 返回NULL
    3. 第二个字符后面的字符串指针 NSGetSizeAndAlignment 
    4. 
    5. 
    6.
    7.
    */
    rac_propertyAttributes *rac_copyPropertyAttributes (objc_property_t property);

3. EXTScop.h


    // 原理 之后解析 使用__attribute((cleanup(function))) 这个修饰符， 离开作用域 执行指定的function
    @onExit 在(goto、break、continue、return之后还能继续执行的) 相当于 final吧 在离开作用域之后运行指定代码

    /*
     遍历使用 rac_weakify_ 宏
     metamacro_foreach_cxt(rac_weakify_,, __weak, __VA_ARGS__)
    */
    @weakify()
    // 当targets or classes 不支持weak时使用这个
    @unsafeify()

    /*
    遍历参数 使用    
    _Pragma("clang diagnostic push") \
    _Pragma("clang diagnostic ignored \"-Wshadow\"") \
    metamacro_foreach(rac_strongify_,, __VA_ARGS__) \
    _Pragma("clang diagnostic pop")

    */

    @strongify()

    rac_weakify

    rac_strongify

    #if DEBUG
    #define rac_keyworkify autoreleasepool({}
    #else
    #define rac_keyworkify try{} @catch (...) {}
    #endif

4. metamacro.h

    所有基本的宏

    // 其实就是逗号表达式 (a, b)
    #define metamacro_exprify(...) \
        ((__VA_ARGS__), true)

    // # + VALUE 的形式会在 VALUE 前后添加双引号，使其成为字符串 使用@可获得NSString 对象
    #define metamacro_stringify(VALUE) \
            metamacro_stringify_(VALUE)

    // metamacro_concat_  最后 = A ## B
    #define metamacro_concat(A, B) \
            metamacro_concat_(A, B)

    // 转换成另一个宏 metamacro_atN(__VA_ARGS__)
    #define metamacro_at(N, ...) \
            metamacro_concat(metamacro_at, N)(__VA_ARGS__)

    // metamacro_at20(__VA_ARGS__ + .....)
    #define metamacro_argcount(...) \
            metamacro_at(20, __VA_ARGS__, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1)

    // (metamacro_foreach_cxt +  metamacro_argcount(__VA_ARGS__))(metamacro_foreach_iter, SEP, MACRO, __VA_ARGS__)
    // 
    #define metamacro_foreach(MACRO, SEP, ...) \
            metamacro_foreach_cxt(metamacro_foreach_iter, SEP, MACRO, __VA_ARGS__)

    /**
    * For each consecutive variadic argument (up to twenty), MACRO is passed the
    * zero-based index of the current argument, CONTEXT, and then the argument
    * itself. The results of adjoining invocations of MACRO are then separated by
    * SEP.
    *
    * Inspired by P99: http://p99.gforge.inria.fr
    */
    // (metamacro_foreach_cxt +  metamacro_argcount(__VA_ARGS__))(MACRO, SEP, CONTEXT, __VA_ARGS__)
    /*
    MACRO(N -1, CONTEXT, __VA_ARGS__(N-1))
                ...
    MACRO(0, CONTEXT, __VA_ARGS__0)
    */
    #define metamacro_foreach_cxt(MACRO, SEP, CONTEXT, ...) \
            metamacro_concat(metamacro_foreach_cxt, metamacro_argcount(__VA_ARGS__))(MACRO, SEP, CONTEXT, __VA_ARGS__)

    // (metamacro_foreach_cxt_recursive + metamacro_argcount(__VA_ARGS__))(MACRO, SEP, CONTEXT, __VA_ARGS__)
    #define metamacro_foreach_cxt_recursive(MACRO, SEP, CONTEXT, ...) \
            metamacro_concat(metamacro_foreach_cxt_recursive, metamacro_argcount(__VA_ARGS__))(MACRO, SEP, CONTEXT, __VA_ARGS__)

    (metamacro_foreach_cxt +  metamacro_argcount(__VA_ARGS__))(metamacro_foreach_concat_iter, SEP, BASE, __VA_ARGS__)
    #define metamacro_foreach_concat(BASE, SEP, ...) \
            metamacro_foreach_cxt(metamacro_foreach_concat_iter, SEP, BASE, __VA_ARGS__)

    // (metamacro_for_cxt+COUNT)(MACRO, SEP, CONTEXT)
    #define metamacro_for_cxt(COUNT, MACRO, SEP, CONTEXT) \
            metamacro_concat(metamacro_for_cxt, COUNT)(MACRO, SEP, CONTEXT)

    /**
    * Returns the first argument given. At least one argument must be provided.
    *
    * This is useful when implementing a variadic macro, where you may have only
    * one variadic argument, but no way to retrieve it (for example, because \c ...
    * always needs to match at least one argument).
    *
    * @code
    // 获取参数的第一个 如果没有提供参数 返回0
    #define varmacro(...) \
        metamacro_head(__VA_ARGS__)

    * @endcode
    */
    #define metamacro_head(...) \
            metamacro_head_(__VA_ARGS__, 0)

    // 返回除了第一个参数外的所有参数 至少提供两个参数
    #define metamacro_tail(...) \
            metamacro_tail_(__VA_ARGS__)

    // (metamacro_take+N)(__VA_ARGS__) 最多20
    #define metamacro_take(N, ...) \
            metamacro_concat(metamacro_take, N)(__VA_ARGS__)

    // (metamacro_drop+N)(__VA_ARGS__) 最多20
    #define metamacro_drop(N, ...) \
            metamacro_concat(metamacro_drop, N)(__VA_ARGS__)

    /**
    * Decrements VAL, which must be a number between zero and twenty, inclusive.
    *
    * This is primarily useful when dealing with indexes and counts in
    * metaprogramming.
    */
    #define metamacro_dec(VAL) \
            metamacro_at(VAL, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19)

    /**
    * Increments VAL, which must be a number between zero and twenty, inclusive.
    *
    * This is primarily useful when dealing with indexes and counts in
    * metaprogramming.
    */
    #define metamacro_inc(VAL) \
            metamacro_at(VAL, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21)

    // metamacro_if_eqA(B)
    #define metamacro_if_eq(A, B) \
            metamacro_concat(metamacro_if_eq, A)(B)

    // metamacro_if_eq_recursiveA(B)
    #define metamacro_if_eq_recursive(A, B) \
            metamacro_concat(metamacro_if_eq_recursive, A)(B)

    // metamacro_atN(1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1)
    #define metamacro_is_even(N) \
            metamacro_at(N, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1)

    // metamacro_atB(1, 0)
    #define metamacro_not(B) \
            metamacro_at(B, 1, 0)

    // IMPLEMENTATION DETAILS FOLLOW!
    // Do not write code that depends on anything below this line.
    #define metamacro_stringify_(VALUE) # VALUE
    #define metamacro_concat_(A, B) A ## B
    #define metamacro_foreach_iter(INDEX, MACRO, ARG) MACRO(INDEX, ARG)
    #define metamacro_head_(FIRST, ...) FIRST
    #define metamacro_tail_(FIRST, ...) __VA_ARGS__
    #define metamacro_consume_(...)
    #define metamacro_expand_(...) __VA_ARGS__

    // implemented from scratch so that metamacro_concat() doesn't end up nesting
    #define metamacro_foreach_concat_iter(INDEX, BASE, ARG) metamacro_foreach_concat_iter_(BASE, ARG)
    #define metamacro_foreach_concat_iter_(BASE, ARG) BASE ## ARG

    // 获取第N个参数
    #define metamacro_at0(...) metamacro_head(__VA_ARGS__)
    #define metamacro_at1(_0, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at2(_0, _1, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at3(_0, _1, _2, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at4(_0, _1, _2, _3, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at5(_0, _1, _2, _3, _4, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at6(_0, _1, _2, _3, _4, _5, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at7(_0, _1, _2, _3, _4, _5, _6, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at8(_0, _1, _2, _3, _4, _5, _6, _7, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at9(_0, _1, _2, _3, _4, _5, _6, _7, _8, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at10(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at11(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at12(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at13(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at14(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at15(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at16(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at17(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at18(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at19(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at20(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19, ...) metamacro_head(__VA_ARGS__)

    // metamacro_foreach_cxt expansions
    #define metamacro_foreach_cxt0(MACRO, SEP, CONTEXT)
    #define metamacro_foreach_cxt1(MACRO, SEP, CONTEXT, _0) MACRO(0, CONTEXT, _0)

    #define metamacro_foreach_cxt2(MACRO, SEP, CONTEXT, _0, _1) \
        metamacro_foreach_cxt1(MACRO, SEP, CONTEXT, _0) \
        SEP \
        MACRO(1, CONTEXT, _1)

    #define metamacro_foreach_cxt3(MACRO, SEP, CONTEXT, _0, _1, _2) \
        metamacro_foreach_cxt2(MACRO, SEP, CONTEXT, _0, _1) \
        SEP \
        MACRO(2, CONTEXT, _2)

    #define metamacro_foreach_cxt4(MACRO, SEP, CONTEXT, _0, _1, _2, _3) \
        metamacro_foreach_cxt3(MACRO, SEP, CONTEXT, _0, _1, _2) \
        SEP \
        MACRO(3, CONTEXT, _3)

    #define metamacro_foreach_cxt5(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4) \
        metamacro_foreach_cxt4(MACRO, SEP, CONTEXT, _0, _1, _2, _3) \
        SEP \
        MACRO(4, CONTEXT, _4)

    #define metamacro_foreach_cxt6(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5) \
        metamacro_foreach_cxt5(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4) \
        SEP \
        MACRO(5, CONTEXT, _5)

    #define metamacro_foreach_cxt7(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6) \
        metamacro_foreach_cxt6(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5) \
        SEP \
        MACRO(6, CONTEXT, _6)

    #define metamacro_foreach_cxt8(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7) \
        metamacro_foreach_cxt7(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6) \
        SEP \
        MACRO(7, CONTEXT, _7)

    #define metamacro_foreach_cxt9(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8) \
        metamacro_foreach_cxt8(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7) \
        SEP \
        MACRO(8, CONTEXT, _8)

    #define metamacro_foreach_cxt10(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9) \
        metamacro_foreach_cxt9(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8) \
        SEP \
        MACRO(9, CONTEXT, _9)

    #define metamacro_foreach_cxt11(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10) \
        metamacro_foreach_cxt10(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9) \
        SEP \
        MACRO(10, CONTEXT, _10)

    #define metamacro_foreach_cxt12(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11) \
        metamacro_foreach_cxt11(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10) \
        SEP \
        MACRO(11, CONTEXT, _11)

    #define metamacro_foreach_cxt13(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12) \
        metamacro_foreach_cxt12(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11) \
        SEP \
        MACRO(12, CONTEXT, _12)

    #define metamacro_foreach_cxt14(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13) \
        metamacro_foreach_cxt13(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12) \
        SEP \
        MACRO(13, CONTEXT, _13)

    #define metamacro_foreach_cxt15(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14) \
        metamacro_foreach_cxt14(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13) \
        SEP \
        MACRO(14, CONTEXT, _14)

    #define metamacro_foreach_cxt16(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15) \
        metamacro_foreach_cxt15(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14) \
        SEP \
        MACRO(15, CONTEXT, _15)

    #define metamacro_foreach_cxt17(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16) \
        metamacro_foreach_cxt16(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15) \
        SEP \
        MACRO(16, CONTEXT, _16)

    #define metamacro_foreach_cxt18(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17) \
        metamacro_foreach_cxt17(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16) \
        SEP \
        MACRO(17, CONTEXT, _17)

    #define metamacro_foreach_cxt19(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18) \
        metamacro_foreach_cxt18(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17) \
        SEP \
        MACRO(18, CONTEXT, _18)

    #define metamacro_foreach_cxt20(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19) \
        metamacro_foreach_cxt19(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18) \
        SEP \
        MACRO(19, CONTEXT, _19)

    // metamacro_foreach_cxt_recursive expansions
    #define metamacro_foreach_cxt_recursive0(MACRO, SEP, CONTEXT)
    #define metamacro_foreach_cxt_recursive1(MACRO, SEP, CONTEXT, _0) MACRO(0, CONTEXT, _0)

    #define metamacro_foreach_cxt_recursive2(MACRO, SEP, CONTEXT, _0, _1) \
        metamacro_foreach_cxt_recursive1(MACRO, SEP, CONTEXT, _0) \
        SEP \
        MACRO(1, CONTEXT, _1)

    #define metamacro_foreach_cxt_recursive3(MACRO, SEP, CONTEXT, _0, _1, _2) \
        metamacro_foreach_cxt_recursive2(MACRO, SEP, CONTEXT, _0, _1) \
        SEP \
        MACRO(2, CONTEXT, _2)

    #define metamacro_foreach_cxt_recursive4(MACRO, SEP, CONTEXT, _0, _1, _2, _3) \
        metamacro_foreach_cxt_recursive3(MACRO, SEP, CONTEXT, _0, _1, _2) \
        SEP \
        MACRO(3, CONTEXT, _3)

    #define metamacro_foreach_cxt_recursive5(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4) \
        metamacro_foreach_cxt_recursive4(MACRO, SEP, CONTEXT, _0, _1, _2, _3) \
        SEP \
        MACRO(4, CONTEXT, _4)

    #define metamacro_foreach_cxt_recursive6(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5) \
        metamacro_foreach_cxt_recursive5(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4) \
        SEP \
        MACRO(5, CONTEXT, _5)

    #define metamacro_foreach_cxt_recursive7(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6) \
        metamacro_foreach_cxt_recursive6(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5) \
        SEP \
        MACRO(6, CONTEXT, _6)

    #define metamacro_foreach_cxt_recursive8(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7) \
        metamacro_foreach_cxt_recursive7(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6) \
        SEP \
        MACRO(7, CONTEXT, _7)

    #define metamacro_foreach_cxt_recursive9(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8) \
        metamacro_foreach_cxt_recursive8(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7) \
        SEP \
        MACRO(8, CONTEXT, _8)

    #define metamacro_foreach_cxt_recursive10(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9) \
        metamacro_foreach_cxt_recursive9(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8) \
        SEP \
        MACRO(9, CONTEXT, _9)

    #define metamacro_foreach_cxt_recursive11(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10) \
        metamacro_foreach_cxt_recursive10(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9) \
        SEP \
        MACRO(10, CONTEXT, _10)

    #define metamacro_foreach_cxt_recursive12(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11) \
        metamacro_foreach_cxt_recursive11(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10) \
        SEP \
        MACRO(11, CONTEXT, _11)

    #define metamacro_foreach_cxt_recursive13(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12) \
        metamacro_foreach_cxt_recursive12(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11) \
        SEP \
        MACRO(12, CONTEXT, _12)

    #define metamacro_foreach_cxt_recursive14(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13) \
        metamacro_foreach_cxt_recursive13(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12) \
        SEP \
        MACRO(13, CONTEXT, _13)

    #define metamacro_foreach_cxt_recursive15(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14) \
        metamacro_foreach_cxt_recursive14(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13) \
        SEP \
        MACRO(14, CONTEXT, _14)

    #define metamacro_foreach_cxt_recursive16(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15) \
        metamacro_foreach_cxt_recursive15(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14) \
        SEP \
        MACRO(15, CONTEXT, _15)

    #define metamacro_foreach_cxt_recursive17(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16) \
        metamacro_foreach_cxt_recursive16(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15) \
        SEP \
        MACRO(16, CONTEXT, _16)

    #define metamacro_foreach_cxt_recursive18(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17) \
        metamacro_foreach_cxt_recursive17(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16) \
        SEP \
        MACRO(17, CONTEXT, _17)

    #define metamacro_foreach_cxt_recursive19(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18) \
        metamacro_foreach_cxt_recursive18(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17) \
        SEP \
        MACRO(18, CONTEXT, _18)

    #define metamacro_foreach_cxt_recursive20(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19) \
        metamacro_foreach_cxt_recursive19(MACRO, SEP, CONTEXT, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18) \
        SEP \
        MACRO(19, CONTEXT, _19)

    // metamacro_for_cxt expansions
    #define metamacro_for_cxt0(MACRO, SEP, CONTEXT)
    #define metamacro_for_cxt1(MACRO, SEP, CONTEXT) MACRO(0, CONTEXT)

    #define metamacro_for_cxt2(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt1(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(1, CONTEXT)

    #define metamacro_for_cxt3(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt2(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(2, CONTEXT)

    #define metamacro_for_cxt4(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt3(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(3, CONTEXT)

    #define metamacro_for_cxt5(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt4(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(4, CONTEXT)

    #define metamacro_for_cxt6(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt5(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(5, CONTEXT)

    #define metamacro_for_cxt7(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt6(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(6, CONTEXT)

    #define metamacro_for_cxt8(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt7(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(7, CONTEXT)

    #define metamacro_for_cxt9(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt8(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(8, CONTEXT)

    #define metamacro_for_cxt10(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt9(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(9, CONTEXT)

    #define metamacro_for_cxt11(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt10(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(10, CONTEXT)

    #define metamacro_for_cxt12(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt11(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(11, CONTEXT)

    #define metamacro_for_cxt13(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt12(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(12, CONTEXT)

    #define metamacro_for_cxt14(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt13(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(13, CONTEXT)

    #define metamacro_for_cxt15(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt14(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(14, CONTEXT)

    #define metamacro_for_cxt16(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt15(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(15, CONTEXT)

    #define metamacro_for_cxt17(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt16(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(16, CONTEXT)

    #define metamacro_for_cxt18(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt17(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(17, CONTEXT)

    #define metamacro_for_cxt19(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt18(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(18, CONTEXT)

    #define metamacro_for_cxt20(MACRO, SEP, CONTEXT) \
        metamacro_for_cxt19(MACRO, SEP, CONTEXT) \
        SEP \
        MACRO(19, CONTEXT)

    // metamacro_if_eq expansions
    #define metamacro_if_eq0(VALUE) \
        metamacro_concat(metamacro_if_eq0_, VALUE)

    #define metamacro_if_eq0_0(...) __VA_ARGS__ metamacro_consume_
    #define metamacro_if_eq0_1(...) metamacro_expand_
    #define metamacro_if_eq0_2(...) metamacro_expand_
    #define metamacro_if_eq0_3(...) metamacro_expand_
    #define metamacro_if_eq0_4(...) metamacro_expand_
    #define metamacro_if_eq0_5(...) metamacro_expand_
    #define metamacro_if_eq0_6(...) metamacro_expand_
    #define metamacro_if_eq0_7(...) metamacro_expand_
    #define metamacro_if_eq0_8(...) metamacro_expand_
    #define metamacro_if_eq0_9(...) metamacro_expand_
    #define metamacro_if_eq0_10(...) metamacro_expand_
    #define metamacro_if_eq0_11(...) metamacro_expand_
    #define metamacro_if_eq0_12(...) metamacro_expand_
    #define metamacro_if_eq0_13(...) metamacro_expand_
    #define metamacro_if_eq0_14(...) metamacro_expand_
    #define metamacro_if_eq0_15(...) metamacro_expand_
    #define metamacro_if_eq0_16(...) metamacro_expand_
    #define metamacro_if_eq0_17(...) metamacro_expand_
    #define metamacro_if_eq0_18(...) metamacro_expand_
    #define metamacro_if_eq0_19(...) metamacro_expand_
    #define metamacro_if_eq0_20(...) metamacro_expand_

    #define metamacro_if_eq1(VALUE) metamacro_if_eq0(metamacro_dec(VALUE))
    #define metamacro_if_eq2(VALUE) metamacro_if_eq1(metamacro_dec(VALUE))
    #define metamacro_if_eq3(VALUE) metamacro_if_eq2(metamacro_dec(VALUE))
    #define metamacro_if_eq4(VALUE) metamacro_if_eq3(metamacro_dec(VALUE))
    #define metamacro_if_eq5(VALUE) metamacro_if_eq4(metamacro_dec(VALUE))
    #define metamacro_if_eq6(VALUE) metamacro_if_eq5(metamacro_dec(VALUE))
    #define metamacro_if_eq7(VALUE) metamacro_if_eq6(metamacro_dec(VALUE))
    #define metamacro_if_eq8(VALUE) metamacro_if_eq7(metamacro_dec(VALUE))
    #define metamacro_if_eq9(VALUE) metamacro_if_eq8(metamacro_dec(VALUE))
    #define metamacro_if_eq10(VALUE) metamacro_if_eq9(metamacro_dec(VALUE))
    #define metamacro_if_eq11(VALUE) metamacro_if_eq10(metamacro_dec(VALUE))
    #define metamacro_if_eq12(VALUE) metamacro_if_eq11(metamacro_dec(VALUE))
    #define metamacro_if_eq13(VALUE) metamacro_if_eq12(metamacro_dec(VALUE))
    #define metamacro_if_eq14(VALUE) metamacro_if_eq13(metamacro_dec(VALUE))
    #define metamacro_if_eq15(VALUE) metamacro_if_eq14(metamacro_dec(VALUE))
    #define metamacro_if_eq16(VALUE) metamacro_if_eq15(metamacro_dec(VALUE))
    #define metamacro_if_eq17(VALUE) metamacro_if_eq16(metamacro_dec(VALUE))
    #define metamacro_if_eq18(VALUE) metamacro_if_eq17(metamacro_dec(VALUE))
    #define metamacro_if_eq19(VALUE) metamacro_if_eq18(metamacro_dec(VALUE))
    #define metamacro_if_eq20(VALUE) metamacro_if_eq19(metamacro_dec(VALUE))

    // metamacro_if_eq_recursive expansions
    #define metamacro_if_eq_recursive0(VALUE) \
        metamacro_concat(metamacro_if_eq_recursive0_, VALUE)

    #define metamacro_if_eq_recursive0_0(...) __VA_ARGS__ metamacro_consume_
    #define metamacro_if_eq_recursive0_1(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_2(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_3(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_4(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_5(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_6(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_7(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_8(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_9(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_10(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_11(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_12(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_13(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_14(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_15(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_16(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_17(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_18(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_19(...) metamacro_expand_
    #define metamacro_if_eq_recursive0_20(...) metamacro_expand_

    #define metamacro_if_eq_recursive1(VALUE) metamacro_if_eq_recursive0(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive2(VALUE) metamacro_if_eq_recursive1(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive3(VALUE) metamacro_if_eq_recursive2(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive4(VALUE) metamacro_if_eq_recursive3(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive5(VALUE) metamacro_if_eq_recursive4(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive6(VALUE) metamacro_if_eq_recursive5(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive7(VALUE) metamacro_if_eq_recursive6(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive8(VALUE) metamacro_if_eq_recursive7(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive9(VALUE) metamacro_if_eq_recursive8(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive10(VALUE) metamacro_if_eq_recursive9(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive11(VALUE) metamacro_if_eq_recursive10(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive12(VALUE) metamacro_if_eq_recursive11(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive13(VALUE) metamacro_if_eq_recursive12(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive14(VALUE) metamacro_if_eq_recursive13(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive15(VALUE) metamacro_if_eq_recursive14(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive16(VALUE) metamacro_if_eq_recursive15(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive17(VALUE) metamacro_if_eq_recursive16(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive18(VALUE) metamacro_if_eq_recursive17(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive19(VALUE) metamacro_if_eq_recursive18(metamacro_dec(VALUE))
    #define metamacro_if_eq_recursive20(VALUE) metamacro_if_eq_recursive19(metamacro_dec(VALUE))

    // metamacro_take expansions
    #define metamacro_take0(...)
    #define metamacro_take1(...) metamacro_head(__VA_ARGS__)
    #define metamacro_take2(...) metamacro_head(__VA_ARGS__), metamacro_take1(metamacro_tail(__VA_ARGS__))
    #define metamacro_take3(...) metamacro_head(__VA_ARGS__), metamacro_take2(metamacro_tail(__VA_ARGS__))
    #define metamacro_take4(...) metamacro_head(__VA_ARGS__), metamacro_take3(metamacro_tail(__VA_ARGS__))
    #define metamacro_take5(...) metamacro_head(__VA_ARGS__), metamacro_take4(metamacro_tail(__VA_ARGS__))
    #define metamacro_take6(...) metamacro_head(__VA_ARGS__), metamacro_take5(metamacro_tail(__VA_ARGS__))
    #define metamacro_take7(...) metamacro_head(__VA_ARGS__), metamacro_take6(metamacro_tail(__VA_ARGS__))
    #define metamacro_take8(...) metamacro_head(__VA_ARGS__), metamacro_take7(metamacro_tail(__VA_ARGS__))
    #define metamacro_take9(...) metamacro_head(__VA_ARGS__), metamacro_take8(metamacro_tail(__VA_ARGS__))
    #define metamacro_take10(...) metamacro_head(__VA_ARGS__), metamacro_take9(metamacro_tail(__VA_ARGS__))
    #define metamacro_take11(...) metamacro_head(__VA_ARGS__), metamacro_take10(metamacro_tail(__VA_ARGS__))
    #define metamacro_take12(...) metamacro_head(__VA_ARGS__), metamacro_take11(metamacro_tail(__VA_ARGS__))
    #define metamacro_take13(...) metamacro_head(__VA_ARGS__), metamacro_take12(metamacro_tail(__VA_ARGS__))
    #define metamacro_take14(...) metamacro_head(__VA_ARGS__), metamacro_take13(metamacro_tail(__VA_ARGS__))
    #define metamacro_take15(...) metamacro_head(__VA_ARGS__), metamacro_take14(metamacro_tail(__VA_ARGS__))
    #define metamacro_take16(...) metamacro_head(__VA_ARGS__), metamacro_take15(metamacro_tail(__VA_ARGS__))
    #define metamacro_take17(...) metamacro_head(__VA_ARGS__), metamacro_take16(metamacro_tail(__VA_ARGS__))
    #define metamacro_take18(...) metamacro_head(__VA_ARGS__), metamacro_take17(metamacro_tail(__VA_ARGS__))
    #define metamacro_take19(...) metamacro_head(__VA_ARGS__), metamacro_take18(metamacro_tail(__VA_ARGS__))
    #define metamacro_take20(...) metamacro_head(__VA_ARGS__), metamacro_take19(metamacro_tail(__VA_ARGS__))

    // metamacro_drop expansions
    #define metamacro_drop0(...) __VA_ARGS__
    #define metamacro_drop1(...) metamacro_tail(__VA_ARGS__)
    #define metamacro_drop2(...) metamacro_drop1(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop3(...) metamacro_drop2(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop4(...) metamacro_drop3(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop5(...) metamacro_drop4(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop6(...) metamacro_drop5(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop7(...) metamacro_drop6(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop8(...) metamacro_drop7(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop9(...) metamacro_drop8(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop10(...) metamacro_drop9(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop11(...) metamacro_drop10(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop12(...) metamacro_drop11(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop13(...) metamacro_drop12(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop14(...) metamacro_drop13(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop15(...) metamacro_drop14(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop16(...) metamacro_drop15(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop17(...) metamacro_drop16(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop18(...) metamacro_drop17(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop19(...) metamacro_drop18(metamacro_tail(__VA_ARGS__))
    #define metamacro_drop20(...) metamacro_drop19(metamacro_tail(__VA_ARGS__))


## files 
1. NSObject+RACSelectorSignal
    
    
    extern NSErrorDomain const RACSelectorSignalErrorDomain

    typedef NS_ERROR_ENUM(RACSelectorSignalErrorDomain, RACSelectorSignalError) {
        RACSelectorSignalErrorMethodSwizzlingRace = 1,
    };

    - (RACSignal<RACTuple *> *)rac_signalForSelector:(SEL)selector;
    - (RACSignal<RACTuple *> *)rac_signalForSelector:(SEL)selector fromProtocol:(Protocol *)protocol;

    //内部方法
    // 返回一个 static NSMutableSet
    static NSMutableSet *swizzledClasses();
    /*
    1. 首先call RACAliasForSelector 获取新的Selector
    2. 根据新的selector通过get_associatedObject 获取RACSubject的实例subject
    3. 获取invacation.target 的class 
    4. 判断class 实例是否响应新的selector 如果响应 invocation的selector = 新的 然后 invkoe
    5. 如果subject 为空 返回新的selector 
    6. subject call sendNext:invocation.rac_argumentTuple 返回yes
    */
    static BOOL RACForwardInvocation(id self, NSInvocation *invocation){}
    /*
    将forwardInvocation替换为自己的实现 交换实现 自己的实现步骤是  call  RACForwardInvocation 
    如果返回yes return 否则 原有实现是否为空 为空doesNotRecognizeSelector 否则 调用原有实现 
    */ 
    static void RACSwizzleForwardInvocation(Class class){}
    /* 原理同上 rac_getImmediateInstanceMethod  自己实现是 rac_getImmediateInstanceMethod 获取目标method 
     如果为空 且为 _objc_msgForward RACAliasForSelector 获取selector  如果objc_getAssociateObject != nil 返回yes
     否则 调用原有的实现
     */
    static void RACSwizzleRespondsToSelector(Class class) {}
    /* 将class 方法替换为自己的实现 自己的实现是返回statedClass */
    static void RACSwizzleGetClass(Class class, Class statedClass) {}
    /*
    将methodSignatureForSelector 替换为自己的实现 自己的实现步骤为
    actclass object_getclass(self) 然后获取selector的method 如果为空 将传入的class的superclass 生成objc_super struct 
    将objc_msgSendSuper 强转为返回 methodSignature * 的方法 然后返回
    */
    static void RACSwizzleMethodSignatureForSelector(Class class) {}
    // 检查返回类型 union struct array 等不支持
    static void RACCheckTypeEncoding(const char * typeEncoding) {}
    /*
    1 首先RACAliasForSelector 
    2 加锁 objc_getAssociatedObject aliasSelector 如果有返回
    3 如果没有  class = RACSwizzleClass(self) 失败返回
    4 创建新的subject 将aliasselector存入associated [self.rac_deallocDisposable addDisposable:[RACDisposable disposableWithBlock:^{
			[subject sendCompleted];
		}]];
    5 获取selector的method 如果 method 为空 如果protocol 为空 RACSignatureForUndefinedSelector 否则 
    struct objc_method_description methodDescription = protocol_getMethodDescription(protocol, selector, NO, YES);
    如果methodDescription的name 是空 methodDescription = protocol_getMethodDescription(protocol, selector, YES, YES);
    6 判断协议中是否有这个方法 有拿到methodDescription的types 否则 NSCAssert RACCheckTypeEncoding 检查typeEncoding
    7 如果都没有 将该函数的实现添加为 _objc_msgForward 返回RACSignal error: 实例
    8 如果method 不为空 且 method 的implementation 不是 _objc_msgForward  获取encoding 检查typeEncoding 如果aliasSelector 已经实现了 替换为_objc_msgForward实现
    */ 
    static RACSignal *NSObjectRACSignalForSelector(NSObject *self, SEL selector, Protocol *protocol) {}
    // 给传入的selector的name加上 rac_alias_ 前缀 返回新的SEL 
    static SEL RACAliasForSelector(SEL originalSelector) {}
    /* sel_getName  s = @"v@:" 然后 然后 : 分割遍历 每出现一次: appendstring: @  */
    static const char *RACSignatureForUndefinedSelector(SEL selector) {}
    /*
    1 statedClass = self.class baseClass = object_getClass(self)
    2  subclass = objc_getAssociatedObject (RACSubclassAssociationKey) 如果 != nil return
    3 如果 baseClass != statedClass 再加锁 swizzledClasses 
    4      如果set里不包含 baseclass 
                RACSwizzleForwardInvocation(baseClass);
				RACSwizzleRespondsToSelector(baseClass);
				RACSwizzleGetClass(baseClass, statedClass);
				RACSwizzleGetClass(object_getClass(baseClass), statedClass);
				RACSwizzleMethodSignatureForSelector(baseClass);
                set add baseclass 
    5 baseclass 加上 _RACSelectorSignal 前缀 转换成Class对象 subclass 如果为空  objc_allocatePair(baseClass, subClassName)
        RACSwizzleForwardInvocation(subclass);
		RACSwizzleRespondsToSelector(subclass);

		RACSwizzleGetClass(subclass, statedClass);
		RACSwizzleGetClass(object_getClass(subclass), statedClass);

		RACSwizzleMethodSignatureForSelector(subclass);

		objc_registerClassPair(subclass);
    6 objc_setclass(self, subclass)  objc_setAssociatedObject(self, RACSubclassAssociationKey, subclass, OBJC_ASSOCIATION_ASSIGN); return
    总结来说就是将对象的class 转换为自己加上前缀的类
    */
    static Class RACSwizzleClass(NSObject *self) {}

    // NSObjectRACSignalForSelector(self, selector, NULL)
    - (RACSignal *)rac_signalForSelector:(SEL)selector {}
    // NSObjectRACSignalForSelector(self, selector, protocol)
    - (RACSignal *)rac_signalForSelector:(SEL)selector fromProtocol:(Protocol *)protocol {}

2. NSObject+RACDeallocting

    // 
    static NSMutableSet *swizzledClasses()
    /*
    把旧的dealloc method swizzle 新的 实现: runtime 拿到RACCompoundDisposable call dispose
    如果原有的dealloc 为空 objc_msgSendSuper 调用父类的 dealloc
    否则 直接调用
    */ 
    static void swizzleDeallocIfNeeded(Class classToSwizzle)
    /*
    通过runtime _cmd 拿到RACSignal 非空返回 否则 创建RACReplaySubject slef.rac_deallocDispose addDisposable
    返回
    */
    - (RACSignal *)rac_willDeallocSignal {}
    /*
    runtime 获取 RACCompoundDisposeable 如果空 call swizzleDeallocIfNeeded 创建RACCompoundDisposable runtime 设置 返回
    */
    - (RACCompoundDisposable *)rac_deallocDisposable 

2. RACSubject <ValueType> : RACSignal<VAlueType> <RACSubscriber>

    // 两个方法
    + (instancetype)subject

    - (void)sendNext:(nullable ValueType)value;

    .m 
    // Contains all current subscribers to the receiver.
    @property (nonatomic, strong, readonly) NSMutableArray *subscribers;

    // Contains all of the receiver's subscriptions to other signals.
    @property (nonatomic, strong, readonly) RACCompoundDisposable *disposable;

    // Enumerates over each of the receiver's `subscribers` and invokes `block` for each.
    - (void)enumerateSubscribersUsingBlock:(void (^)(id<RACSubscriber> subscriber))block;

    - (instancetype)init { _disposable = [RACCompoundDisposable compoundDisposable] }

    - (void)dealloc { self.disposable dispose }

    - (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {}

3. RACSignal <RACStream>

    + (RACSignal<ValueType> *)createSignal:(RACDisposable * _Nullable (^)(id<RACSubscriber> subscriber))didSubscribe RAC_WARN_UNUSED_RESULT;

    
    + (RACSignal<ValueType> *)error:(nullable NSError *)error RAC_WARN_UNUSED_RESULT;

    
    + (RACSignal<ValueType> *)never RAC_WARN_UNUSED_RESULT;

    
    + (RACSignal<ValueType> *)startEagerlyWithScheduler:(RACScheduler *)scheduler block:(void (^)(id<RACSubscriber> subscriber))block;

    + (RACSignal<ValueType> *)startLazilyWithScheduler:(RACScheduler *)scheduler block:(void (^)(id<RACSubscriber> subscriber))block RAC_WARN_UNUSED_RESULT;

    @end

    @interface RACSignal<__covariant ValueType> (RACStream)


    + (RACSignal<ValueType> *)return:(nullable ValueType)value RAC_WARN_UNUSED_RESULT;

    
    + (RACSignal<ValueType> *)empty RAC_WARN_UNUSED_RESULT;

    
    typedef RACSignal * _Nullable (^RACSignalBindBlock)(ValueType _Nullable value, BOOL *stop);

    
    - (RACSignal *)bind:(RACSignalBindBlock (^)(void))block RAC_WARN_UNUSED_RESULT;

    
    - (RACSignal *)concat:(RACSignal *)signal RAC_WARN_UNUSED_RESULT;

    - (RACSignal<RACTwoTuple<ValueType, id> *> *)zipWith:(RACSignal *)signal RAC_WARN_UNUSED_RESULT;

    @end

    @interface RACSignal<__covariant ValueType> (RACStreamOperations)

    - (RACSignal *)flattenMap:(__kindof RACSignal * _Nullable (^)(ValueType _Nullable value))block RAC_WARN_UNUSED_RESULT;

    - (RACSignal *)flatten RAC_WARN_UNUSED_RESULT;

    - (RACSignal *)map:(id _Nullable (^)(ValueType _Nullable value))block RAC_WARN_UNUSED_RESULT;

    - (RACSignal *)mapReplace:(nullable id)object RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)filter:(BOOL (^)(ValueType _Nullable value))block RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)ignore:(nullable ValueType)value RAC_WARN_UNUSED_RESULT;

    - (RACSignal *)reduceEach:(RACReduceBlock)reduceBlock RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)startWith:(nullable ValueType)value RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)skip:(NSUInteger)skipCount RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)take:(NSUInteger)count RAC_WARN_UNUSED_RESULT;

    + (RACSignal<RACTuple *> *)zip:(id<NSFastEnumeration>)signals RAC_WARN_UNUSED_RESULT;

    + (RACSignal<ValueType> *)zip:(id<NSFastEnumeration>)signals reduce:(RACGenericReduceBlock)reduceBlock RAC_WARN_UNUSED_RESULT;

    + (RACSignal<ValueType> *)concat:(id<NSFastEnumeration>)signals RAC_WARN_UNUSED_RESULT;

    - (RACSignal *)scanWithStart:(nullable id)startingValue reduce:(id _Nullable (^)(id _Nullable running, ValueType _Nullable next))reduceBlock RAC_WARN_UNUSED_RESULT;

    - (RACSignal *)scanWithStart:(nullable id)startingValue reduceWithIndex:(id _Nullable (^)(id _Nullable running, ValueType _Nullable next, NSUInteger index))reduceBlock RAC_WARN_UNUSED_RESULT;

    - (RACSignal *)combinePreviousWithStart:(nullable ValueType)start reduce:(id _Nullable (^)(ValueType _Nullable previous, ValueType _Nullable current))reduceBlock RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)takeUntilBlock:(BOOL (^)(ValueType _Nullable x))predicate RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)takeWhileBlock:(BOOL (^)(ValueType _Nullable x))predicate RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)skipUntilBlock:(BOOL (^)(ValueType _Nullable x))predicate RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)skipWhileBlock:(BOOL (^)(ValueType _Nullable x))predicate RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)distinctUntilChanged RAC_WARN_UNUSED_RESULT;

    @end

    @interface RACSignal<__covariant ValueType> (Subscription)

    - (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber;

    - (RACDisposable *)subscribeNext:(void (^)(ValueType _Nullable x))nextBlock;

    - (RACDisposable *)subscribeNext:(void (^)(ValueType _Nullable x))nextBlock completed:(void (^)(void))completedBlock;

    - (RACDisposable *)subscribeNext:(void (^)(ValueType _Nullable x))nextBlock error:(void (^)(NSError * _Nullable error))errorBlock completed:(void (^)(void))completedBlock;

    - (RACDisposable *)subscribeError:(void (^)(NSError * _Nullable error))errorBlock;

    - (RACDisposable *)subscribeCompleted:(void (^)(void))completedBlock;

    - (RACDisposable *)subscribeNext:(void (^)(ValueType _Nullable x))nextBlock error:(void (^)(NSError * _Nullable error))errorBlock;

    - (RACDisposable *)subscribeError:(void (^)(NSError * _Nullable error))errorBlock completed:(void (^)(void))completedBlock;

    @end

    @interface RACSignal<__covariant ValueType> (Debugging)

    - (RACSignal<ValueType> *)logAll RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)logNext RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)logError RAC_WARN_UNUSED_RESULT;

    - (RACSignal<ValueType> *)logCompleted RAC_WARN_UNUSED_RESULT;

    @end

    @interface RACSignal<__covariant ValueType> (Testing)

    - (nullable ValueType)asynchronousFirstOrDefault:(nullable ValueType)defaultValue success:(nullable BOOL *)success error:(NSError * _Nullable * _Nullable)error timeout:(NSTimeInterval)timeout;

    - (nullable ValueType)asynchronousFirstOrDefault:(nullable ValueType)defaultValue success:(nullable BOOL *)success error:(NSError * _Nullable * _Nullable)error;

    - (BOOL)asynchronouslyWaitUntilCompleted:(NSError * _Nullable * _Nullable)error timeout:(NSTimeInterval)timeout;

    - (BOOL)asynchronouslyWaitUntilCompleted:(NSError * _Nullable * _Nullable)error;

    @end


4. RACStream <__covariant ValueType> : NSobject
    // __covariant 协变 子类转父类 __contravariant 逆变 父类转子类
    // 这个类可以理解为一个集合

    // 生成一个kong的stream  抽象方法
    + (__kindof RACStream<ValueType> *)empty;

    /*
    传入一个参数 生成一个只包含该参数的集合 stream   抽象方法: 需要子类来实现 自己的实现是抛出异常
    */
    + (__kindof RACStream<ValueType> *)return:(nullable ValueType)value;

    // 稍后解析  传入一个value 返回一个新的同样的stream class 
    typedef RACStream * _Nullable (^RACStreamBindBlock)(ValueType _Nullable value, BOOL *stop);

    //*
     抽象方法
    */
    - (__kindof RACStream *)bind:(RACStreamBindBlock (^)(void))block;

    /*
    抽象方法 相当于addObjectFromArray吧
    */
    - (__kindof RACStream *)concat:(RACStream *)stream;

    /*
    抽象方法
    */
    - (__kindof RACStream *)zipWith:(RACStream *)stream;

    @end

    @interface RACStream ()

    @property (copy) NSString *name;
    - (instancetype)setNameWithFormat:(NSString *)format, ... NS_FORMAT_FUNCTION(1, 2);

    @end

    @interface RACStream<__covariant ValueType> (Operations)
    // 判断block返回的stream是否为空 如果是空创建一个empty的stream 然后call  bind 在setNameWithFormat: 
    - (__kindof RACStream *)flattenMap:(__kindof RACStream * _Nullable (^)(ValueType _Nullable value))block;
    // call self flattenMap:
    - (__kindof RACStream *)flatten;

    - (__kindof RACStream *)map:(id _Nullable (^)(ValueType _Nullable value))block;

    - (__kindof RACStream *)mapReplace:(nullable id)object;

    - (__kindof RACStream<ValueType> *)filter:(BOOL (^)(ValueType _Nullable value))block;

    - (__kindof RACStream<ValueType> *)ignore:(nullable ValueType)value;

    - (__kindof RACStream *)reduceEach:(RACReduceBlock)reduceBlock;

    - (__kindof RACStream<ValueType> *)startWith:(nullable ValueType)value;

    - (__kindof RACStream<ValueType> *)skip:(NSUInteger)skipCount;

    - (__kindof RACStream<ValueType> *)take:(NSUInteger)count;

    + (__kindof RACStream<ValueType> *)zip:(id<NSFastEnumeration>)streams;

    + (__kindof RACStream<ValueType> *)zip:(id<NSFastEnumeration>)streams reduce:(RACGenericReduceBlock)reduceBlock;

    + (__kindof RACStream<ValueType> *)concat:(id<NSFastEnumeration>)streams;

    - (__kindof RACStream *)scanWithStart:(nullable id)startingValue reduce:(id _Nullable (^)(id _Nullable running, ValueType _Nullable next))reduceBlock;

    - (__kindof RACStream *)scanWithStart:(nullable id)startingValue reduceWithIndex:(id _Nullable (^)(id _Nullable running, ValueType _Nullable next, NSUInteger index))reduceBlock;

    - (__kindof RACStream *)combinePreviousWithStart:(nullable ValueType)start reduce:(id _Nullable (^)(ValueType _Nullable previous, ValueType _Nullable current))reduceBlock;

    - (__kindof RACStream<ValueType> *)takeUntilBlock:(BOOL (^)(ValueType _Nullable x))predicate;

    - (__kindof RACStream<ValueType> *)takeWhileBlock:(BOOL (^)(ValueType _Nullable x))predicate;

    - (__kindof RACStream<ValueType> *)skipUntilBlock:(BOOL (^)(ValueType _Nullable x))predicate;

    - (__kindof RACStream<ValueType> *)skipWhileBlock:(BOOL (^)(ValueType _Nullable x))predicate;

    - (__kindof RACStream<ValueType> *)distinctUntilChanged;



2. MKAnnotationView (RACSignalSupport)
    /*
    通过runtime 创建和获取  创建过程
    1 rac_signalForSelector @selector(prepearForReuse)
    2 mapReplace: RACUnit.defaultUnit
    3  setNameWithFormat
    */
    @property (nonatomic, strong, readonly) RACSignal<RACUnit *> *rac_prepareForReuseSignal;

3. RACDisposable 

    //
    @property (atomic, assign, getter = isDisposed, readonly) BOOL disposed;
    
    /*
    
    */
    + (instancetype)disposableWithBlock:(void (^)(void))block;

    /*
    while 循环判断 临时变量blockPtr和 _disposeBlock 是否相等 若相等 _disposeBlock 置 NULL 如果blockPtr != self  disposeBlock = blockPtr 
    运行disposeBlock
     OSAtomicCompareAndSwapPtrBarrier 交换 临时变量 blockPtr 和 _disposeBlock
    释放旧的 执行新的 https://www.jianshu.com/p/940544653ede
    CFBridgingRelease 把非OC的指针指向OC并转换为ARC
    */
    - (void)dispose;

    /*  */
    - (RACScopedDisposable *)asScopedDisposable{ return [RACScopedDisposable scopedDisposableWithDisposable:self]; };

    内部方法
    // volatile 每次从内存中读取
    { void * volatile _disposeBlock; }

    // 通过判断 成员变量是否为空
    - (BOOL)isDisposed { return _disposeBlock == NULL; }

    /* 初始化变量 OSMemoryBarrier 防止内存乱序访问 
    https://blog.csdn.net/world_hello_100/article/details/50131497
    */
    init{ _disposeBlock = (__bridge void *)self; OSMemoryBarrier(); };

    - (instancetype)initWithBlock:(void (^)(void))block { _disposeBlock = (void *)CFBridgingRetain([block copy]); OSMemoryBarrier(); }
    // 如果 _disposeBlock 为空 或者等于self return 否则释放
    - (void)dealloc;

4. RACScopedDisposable  : RACDisposable
    
    /* 唯一的实例化方法 */
    + (instancetype)scopedDisposableWithDisposable:(RACDisposable *)disposable;
    
    // 调用父类的 disposablWithBlock 在block里调用 disposable dispose
    + (instancetype)scopedDisposableWithDisposable:(RACDisposable *)disposable {}

    // 调用dispose
    - (void)dealloc {}
    // 
    - (RACScopedDisposable *)asScopedDisposable { return self; }

5. RACCompoundDisposable : RACDisposable

    + (instancetype)compoundDisposable;

    + (instancetype)compoundDisposableWithDisposables:(nullable NSArray *)disposables;

    - (void)addDisposable:(nullable RACDisposable *)disposable;

    - (void)removeDisposable:(nullable RACDisposable *)disposable;

    .m

    #define RACCompoundDisposableInlineCount 2

    // 创建一个 CFMutableArray
    static CFMutableArrayRef RACCreateDisposablesArray(void) {
        // Compare values using only pointer equality.
        CFArrayCallBacks callbacks = kCFTypeArrayCallBacks;
        callbacks.equal = NULL;

        return CFArrayCreateMutable(NULL, 0, &callbacks);
    }
    // 实例变量
    {
        pthread_mutex_t _mutex;
        #if RACCompoundDisposableInlineCount
        RACDisposable *_inlineDisposables[RACCompoundDisposableInlineCount];
        #endif
        CFMutableArrayRef _disposables;
        BOOL _disposed;
    }
    // 加锁 在锁的时候临时变量=_disposed 返回
    - (BOOL)isDisposed {}

    #pragma mark Lifecycle

    + (instancetype)compoundDisposable {}

    + (instancetype)compoundDisposableWithDisposables:(NSArray *)disposables {return [[self alloc] initWithDisposables:disposables];}
    
    // 初始化_mutex
    - (instancetype)init {}

    /*
    遍历 otherDisposables 放入 _inlineDisposables 最多放入 RACCompoundDisposableInlineCount
    如果超过限制个数  _disposables = RACCreateDisposablesArray 把剩余的加入到数组里
    */ 
    - (instancetype)initWithDisposables:(NSArray *)otherDisposables {}

    - (instancetype)initWithBlock:(void (^)(void))block {
        RACDisposable *disposable = [RACDisposable disposableWithBlock:block];
        return [self initWithDisposables:@[ disposable ]];
    }

    /*
    置空_inlineDisposables  和释放 _disposables destory _mutex
     */
    - (void)dealloc {}

    #pragma mark Addition and Removal
    /*
    如果disposable == nil 或者 disposed 返回 加锁  如果 _disposed  disposable dispose
    否则  _inlineDisposable 哪个是空的  disposable就存入空的位置 跳出 释放锁 如果_disposable == NULL create append
    */
    - (void)addDisposable:(RACDisposable *)disposable {}
   /*
    加锁  ！_disposable  _inline 中如果有disposable 置空 如果_disposables != NULL 遍历 _disposables 如果 == disposable remmove
    */ 
    - (void)removeDisposable:(RACDisposable *)disposable {}

    #pragma mark RACDisposable

    // 将value 转换成disposable 然后 dispose
    static void disposeEach(const void *value, void *context) {}
    /*
    临时变量 remainings CFArrayRef 加锁 _disposed = yes _inlines 遍历置空
    remainmings = _disposables _disposables = NULL 解锁 inlineCopy dispose
    如果 remainings 非空 CFArrayAppluFunction  disposeEach CFRelease
    */
    - (void)dispose {}

6. RACSubscriber

    .m 

    // These callbacks should only be accessed while synchronized on self.
    @property (nonatomic, copy) void (^next)(id value);
    @property (nonatomic, copy) void (^error)(NSError *error);
    @property (nonatomic, copy) void (^completed)(void);

    @property (nonatomic, strong, readonly) RACCompoundDisposable *disposable;

    /*
    创建一个新的subscriber  next error completed 一一赋值
    */
    + (instancetype)subscriberWithNext:(void (^)(id x))next error:(void (^)(NSError *error))error completed:(void (^)(void))completed {}
    /*
    创建一个RACDisposable对象 在block中将自己的next error completed 置空 将disposable对象加入到_disposable中
    */
    - (instancetype)init {}

    - (void)dealloc {   [self.disposable dispose];   }

    #pragma mark RACSubscriber
    /* 同步锁 copy next 调用copy */
    - (void)sendNext:(id)value {}
    /* 同步锁 copy eroor 调用copy */
    - (void)sendError:(NSError *)e {}
    /* 同步锁 block = _completed copy _disposable dispose 如果block非空 调block */
    - (void)sendCompleted {}
    /*
    如果other disposed return else self.disposable add 然后other adddisposable 在block中 self.disposable removedisposable
    */
    - (void)didSubscribeWithDisposable:(RACCompoundDisposable *)otherDisposable {}

7. RACPassthroughSubscriber

    // Returns an initialized passthrough subscriber.
    - (instancetype)initWithSubscriber:(id<RACSubscriber>)subscriber signal:(RACSignal *)signal disposable:(RACCompoundDisposable *)disposable; 
    
    // 将'//s+' 替换为空格'' 然后转换成char *
    static const char *cleanedDTraceString(NSString *original) {}
    // 判断signal的description中有没有" name:" 如果有 替换为空  cleanedDTraceString
    static const char *cleanedSignalDescription(RACSignal *signal) {}

    #endif

    @property (nonatomic, strong, readonly) id<RACSubscriber> innerSubscriber;
    @property (nonatomic, unsafe_unretained, readonly) RACSignal *signal;
    @property (nonatomic, strong, readonly) RACCompoundDisposable *disposable;

    /*
    _innerSubscriber _signal _dispoasble 一一赋值  innersubscriber didSubscribeWithDisposable:
    */
    - (instancetype)initWithSubscriber:(id<RACSubscriber>)subscriber signal:(RACSignal *)signal disposable:(RACCompoundDisposable *)disposable {    }
    // _innerSubscriber sendNext:
    - (void)sendNext:(id)value {
        // 这段代码是DTrace 生成的一段用于debug的吧 http://www.brendangregg.com/dtrace.html 有空了可以研究一下
        if (RACSIGNAL_NEXT_ENABLED()) {
            RACSIGNAL_NEXT(cleanedSignalDescription(self.signal), cleanedDTraceString(self.innerSubscriber.description), cleanedDTraceString([value description]));
        } }

    //  _innerSubscriber sendError:
    - (void)sendError:(NSError *)error {
        if (RACSIGNAL_ERROR_ENABLED()) {
            RACSIGNAL_ERROR(cleanedSignalDescription(self.signal), cleanedDTraceString(self.innerSubscriber.description), cleanedDTraceString(error.description));
        }
    }
    //  _innerSubscriber sendCompleted:
    - (void)sendCompleted {
        if (RACSIGNAL_COMPLETED_ENABLED()) {
            RACSIGNAL_COMPLETED(cleanedSignalDescription(self.signal), cleanedDTraceString(self.innerSubscriber.description));
        }
    }
    // 如果disposable 不是自己disposable addDisposable 
    - (void)didSubscribeWithDisposable:(RACCompoundDisposable *)disposable {
        if (disposable != self.disposable) {
            [self.disposable addDisposable:disposable];
        }
    }

7. RACDynamicSignal

    .m
    @property (nonatomic, copy, readonly) RACDisposable * (^didSubscribe)(id<RACSubscriber> subscriber);
    // 新建一个DynamicSignal 实例 然后_didSuscriber = subscriber setNameForFormat
    + (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe {}
    /*
    新建RACCompoundDisposable 然后新建一个RACPassthroughSubscriber
    如果didSubscribe 非空 新建一个 RACDisposable 通过RACScheduler.subscriptionScheduler schedule:
    在block内 通过didsubscriber(subscriber) 拿到innerDisposable disposable add inner
    在block外 disposable addDisposable 一开始新建的RACCompoundDisposable
    */
    - (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {}

8. RACEmptySignal

9. RACErrorSignal

    // The error to send upon subscription.
    @property (nonatomic, strong, readonly) NSError *error;

    // 创建一个新的RACErrorSignal signal的error = error 在DEBUG的时候 setName 把error拼上 否则为方法名
    + (RACSignal *)error:(NSError *)error {}
    // RACScheduler.subscriptionScheduler scheduler 在block中subscriber sendError: self.error
    - (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {}


10. RACReturnSignal


@interface RACReturnSignal ()

// The value to send upon subscription.
@property (nonatomic, strong, readonly) id value;

@end

@implementation RACReturnSignal

#pragma mark Properties

// Only allow this signal's name to be customized in DEBUG, since it's
// potentially a singleton in release builds (see +return:).
- (void)setName:(NSString *)name {
#ifdef DEBUG
	[super setName:name];
#endif
}

- (NSString *)name {
#ifdef DEBUG
	return super.name;
#else
	return @"+return:";
#endif
}

#pragma mark Lifecycle

+ (RACSignal *)return:(id)value {
#ifndef DEBUG
	// In release builds, use singletons for two very common cases.
	if (value == RACUnit.defaultUnit) {
		static RACReturnSignal *unitSingleton;
		static dispatch_once_t unitPred;

		dispatch_once(&unitPred, ^{
			unitSingleton = [[self alloc] init];
			unitSingleton->_value = RACUnit.defaultUnit;
		});

		return unitSingleton;
	} else if (value == nil) {
		static RACReturnSignal *nilSingleton;
		static dispatch_once_t nilPred;

		dispatch_once(&nilPred, ^{
			nilSingleton = [[self alloc] init];
			nilSingleton->_value = nil;
		});

		return nilSingleton;
	}
#endif

	RACReturnSignal *signal = [[self alloc] init];
	signal->_value = value;

#ifdef DEBUG
	[signal setNameWithFormat:@"+return: %@", value];
#endif

	return signal;
}

#pragma mark Subscription

- (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {
	NSCParameterAssert(subscriber != nil);

	return [RACScheduler.subscriptionScheduler schedule:^{
		[subscriber sendNext:self.value];
		[subscriber sendCompleted];
	}];
}


 RACReturnSignal<__covariant ValueType> : RACSignal<ValueType>

+ (RACSignal<ValueType> *)return:(ValueType)value;

11. RACMulticastConnection.h <__covariant ValueType> : NSObject

    /// The multicasted signal.
    @property (nonatomic, strong, readonly) RACSignal<ValueType> *signal;
    - (RACDisposable *)connect;
    - (RACSignal<ValueType> *)autoconnect RAC_WARN_UNUSED_RESULT;

    .m
    @interface RACMulticastConnection () {
        RACSubject *_signal;
        int32_t volatile _hasConnected;
    }
    // 源信号
    @property (nonatomic, readonly, strong) RACSignal *sourceSignal;
    // 序列释放
    @property (strong) RACSerialDisposable *serialDisposable;
    @end
    // 一一赋值 然后 初始化_serialDisposable
    - (instancetype)initWithSourceSignal:(RACSignal *)source subject:(RACSubject *)subject { }
    /*
    OSAtomicCompareAndSwap32Barrier 参数顺序 OldValue newValue theValue 在内存中比较oldValue 和 the value 如果相同 把new value 存入内存中
    如果相同 return True else false 
    如果_hasConnect 为NO self.serialDisposable.disposable = [self.sourceSignal subscribe:_signal]; 返回 self.serialDisposable
    */
    - (RACDisposable *)connect {}
    /*
    一个局部变量 __block volatile int32_t subscriberCount = 0;
    return [[RACSignal createSignal] setNameWithFormat] 在block内  OSAtomicIncrement32Barrier subCount
    RACDisposable *subDis = self.signal subscribe:subscriber; conDis = self connect;
    return RACDisposable disposableWithBlock 在这个block内 subDis dispose 如果OSAtomicDecrement32Barrier(&subscriberCount) == 0 conDis dispose;
    */ 
    - (RACSignal *)autoconnect {}

 
12. RACReplySubject <ValueType> : RACSubject<ValueType>

    // self alloc initWithCapacity:
    + (instancetype)replaySubjectWithCapacity:(NSUInteger)capacity;

    .m 
    const NSUInteger RACReplaySubjectUnlimitedCapacity = NSUIntegerMax;

    @property (nonatomic, assign, readonly) NSUInteger capacity;

    // These properties should only be modified while synchronized on self.
    @property (nonatomic, strong, readonly) NSMutableArray *valuesReceived;
    @property (nonatomic, assign) BOOL hasCompleted;
    @property (nonatomic, assign) BOOL hasError;
    @property (nonatomic, strong) NSError *error;

    - (instancetype)init {}
    // 给_capacity 赋值 然后判断capacity是否等于 NSUIntegerMax 如果是 _valuesReceived = NSMutableArray array 否则 = arrayWithCapacity
    - (instancetype)initWithCapacity:(NSUInteger)capacity {}
    /*
    新建一个RACCompoundDisposable变量 comDis 然后通过RACScheduler.subscriptionScheduler scheduler 创建 RACDipsoable schDis 在block内
    同步锁 遍历自己的valuesReceived  如果comDis disposed return 然后subscriber sendNext value == RACTupleNil.tupleNil ? nil : value
    再次检查comDis disposed 如果self.hascompleted sendCompleted else 如果 self.hasError sendError  else 
    RACDisposable subsDis = super subscribe:subscriber  comDis addDisposable:subsDis
    最后 comDis addDisposable:schDis return
    */ 
    - (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber { }
    /*
    同步锁 self.valuesReceived addObject value 或者RACTupleNil.tupleNil super sendNext:value 
    如果 capacity != 最大 并且 valuesReceived.count > capacity 把最前面的移除
    */ 
    - (void)sendNext:(id)value { }
    // 同步锁 hasCompleted 设为YES 然后 super sendCompleted
    - (void)sendCompleted { }
    // 同步锁 同上
    - (void)sendError:(NSError *)e { }

13. RACScheduler 
    // 安排者 决定什么时候什么地方去执行相应的工作

    // 将队列的优先级转换为自定义枚举
    typedef enum : long {
        RACSchedulerPriorityHigh = DISPATCH_QUEUE_PRIORITY_HIGH,
        RACSchedulerPriorityDefault = DISPATCH_QUEUE_PRIORITY_DEFAULT,
        RACSchedulerPriorityLow = DISPATCH_QUEUE_PRIORITY_LOW,
        RACSchedulerPriorityBackground = DISPATCH_QUEUE_PRIORITY_BACKGROUND,
    } RACSchedulerPriority;

    typedef void (^RACSchedulerRecursiveBlock)(void (^reschedule)(void));
            
    // 返回以个static RACImmediatedScheduler
    + (RACScheduler *)immediateScheduler;
    // 返回一个static RACTargetQueueScheduler initWithName:@"org.reactivecocoa.ReactiveObjC.RACScheduler.mainThreadScheduler" queue:main_queue
    + (RACScheduler *)mainThreadScheduler;

    // RACTargetQueueScheduler initWithName:name queue:get_global_queue(priority)
    + (RACScheduler *)schedulerWithPriority:(RACSchedulerPriority)priority name:(nullable NSString *)name;

    // 调用上面的方法
    + (RACScheduler *)schedulerWithPriority:(RACSchedulerPriority)priority;

    // schedulerWithPriority:default
    + (RACScheduler *)scheduler;

    // 获取当前线程的scheduler 非空返回 否则 如果当前在主线程  返回mainThreadSecheduler 返回nil
    + (nullable RACScheduler *)currentScheduler;

    // 抽象方法 由子类实现 block 不能为空 返回的Disposable 可用在调用之前取消 如果取消不支持 返回nil
    - (nullable RACDisposable *)schedule:(void (^)(void))block;

    // 抽象方法 由子类实现 在某个时间或之后 执行 在immediateScheduler 中调用时 线程会被阻塞直到时间到达
    // block 不能为空 返回的Disposable 可用在调用之前取消 如果取消不支持 返回nil
    - (nullable RACDisposable *)after:(NSDate *)date schedule:(void (^)(void))block;
    // 实际是调用上面的方法
    - (nullable RACDisposable *)afterDelay:(NSTimeInterval)delay schedule:(void (^)(void))block;
    // 抽象方法 由子类实现 
    - (nullable RACDisposable *)after:(NSDate *)date repeatingEvery:(NSTimeInterval)interval withLeeway:(NSTimeInterval)leeway schedule:(void (^)(void))block;

    // 新建一个RACCompoundDisposable  scheduleRecursiveBlock:[recursiveBlock copy] addingToDisposable
    - (nullable RACDisposable *)scheduleRecursiveBlock:(RACSchedulerRecursiveBlock)recursiveBlock;

    // .m
    // The key for the thread-specific current scheduler.
    NSString * const RACSchedulerCurrentSchedulerKey = @"RACSchedulerCurrentSchedulerKey";

    @property (nonatomic, readonly, copy) NSString *name;
    #pragma mark NSObject
    // 输出自己的类名 指针 和自己的name
    - (NSString *)description {}
    // 如果name为空  则默认为@"org.reactivecocoa.ReactiveObjC.%@.anonymousScheduler", self.class
    - (instancetype)initWithName:(NSString *)name {    }
    // 返回一个static RACSubscriptionScheduler
    + (RACScheduler *)subscriptionScheduler {}

    /*
    新建一个 selfDis = RACCompoundDisposable disposable addDis:selfDis 听过self scheduler: 新建一个 scDis = RACDisposable 
    在block内 disposable remove:selfDis 如果disposable disposed return 一个schblock 在schblock内递归调用自己
    NSLock lock  一个NSUInteger reschCount = 0 BOOL reschImmediate = NO;
    执行 recursiveBlock 在block内  加锁 BOOL immediate = reschImmediate 如果是NO reschCount++ 解锁 如果是yes 调用schblock
    加锁 syncCount = reschCount reschImmediate = yes 解锁 for syncCount 调用schblock 最后selfDis addDisposable schDis
    */
    - (void)scheduleRecursiveBlock:(RACSchedulerRecursiveBlock)recursiveBlock addingToDisposable:(RACCompoundDisposable *)disposable {}
    /*
    获取pre = currentScheduler 将当前线程的scheduler == self 调用block  如果pre 非空在放回当前线程的scheduler 否则 删除当前线程的scheduler
    */
    - (void)performAsCurrentScheduler:(void (^)(void))block { }

14. RACSerialDisposable:RACDisposable

    @property (atomic, strong, nullable) RACDisposable *disposable;

    + (instancetype)serialDisposableWithDisposable:(nullable RACDisposable *)disposable;

    - (nullable RACDisposable *)swapInDisposable:(nullable RACDisposable *)newDisposable;

    .m
    {
        RACDisposable * _disposable;
        BOOL _disposed;
        pthread_mutex_t _mutex;
    }
    // 加锁 BOOL 常量 disposed = _disposed 解锁 return
    - (BOOL)isDisposed {}
    // 加锁 BOOL 常量 result = _disposable 解锁 return
    - (RACDisposable *)disposable {    }
    // swapInDisposable
    - (void)setDisposable:(RACDisposable *)disposable { }
    // 新建一个RACSerialDisposable 将他的disposable 赋值为 disposable 返回
    + (instancetype)serialDisposableWithDisposable:(RACDisposable *)disposable { }
    // 初始化 _mutex
    - (instancetype)init { }
    // self.disposable = [RACDisposable disposableWithBlock:block];
    - (instancetype)initWithBlock:(void (^)(void))block { }
    // 销毁 _mutex
    - (void)dealloc { }
    /*
     变量RACDisposable exitDis BOOL alrDis 加锁 alrDis = _disposable 如果 exitDis == NO exitDis = _disposable
     _disposable = newDisposable 解锁 如果 alrDis == YES newDisposable dispose return exitDis
    */
    - (RACDisposable *)swapInDisposable:(RACDisposable *)newDisposable {}
    // RACDisposable exitDis  加锁 如果 !_disposable exitDis = YES _disposable = nil; 解锁 exitDis dispose
    - (void)dispose { }

15. RACTuple

    /*
    给定一些值 创建一个RSCTuple  value 不能为nil
    ([RACTuplePack_class_name(__VA_ARGS__) tupleWithObjectsFromArray:@[ metamacro_foreach(RACTuplePack_object_or_ractuplenil,, __VA_ARGS__) ]])
    metamacro_foreach_cxt(metamacro_foreach_iter, nil, RACTuplePack_object_or_ractuplenil, __VA_ARGS__)
    metamacro_concat(metamacro_foreach_cxt, metamacro_argcount(__VA_ARGS__))(metamacro_foreach_iter, SEP, RACTuplePack_object_or_ractuplenil, __VA_ARGS__)
    metamacro_foreach_cxt20)(metamacro_foreach_iter, SEP, RACTuplePack_object_or_ractuplenil, __VA_ARGS__)
    metamacro_foreach_iter(19, RACTuplePack_object_or_ractuplenil, _19)
    RACTuplePack_object_or_ractuplenil(19, _19);
    (_19) ?: RACTupleNil.tupleNil,
    [RACOneTuple|  ]
    */ 
    #define RACTuplePack(...) \
        RACTuplePack_(__VA_ARGS__)

    /*
    metamacro_foreach(RACTupleUnpack_decl,, __VA_ARGS__)
    metamacro_foreach_cxt(metamacro_foreach_iter, SEP, RACTupleUnpack_decl, __VA_ARGS__)
    metamacro_concat(metamacro_foreach_cxt, metamacro_argcount(__VA_ARGS__))(MACRO, SEP, CONTEXT, __VA_ARGS__)
    metamacro_foreach_cxt20(metamacro_foreach_iter, SEP, RACTupleUnpack_decl, __VA_ARGS__)
    metamacro_foreach_iter(19, RACTupleUnpack_decl, _19)
    RACTupleUnpack_decl(19, _19);
    __strong id RACTupleUnpack_decl_name(19)
    metamacro_concat(metamacro_concat(RACTupleUnpack, __LINE__), metamacro_concat(_var, INDEX))
    metamacro_concat(RACTupleUnpack ## __LINE__, metamacro_concat(_var, INDEX))
    RACTupleUnpack ## __LINE__ ## _var ## 19
    */ 
    #define RACTupleUnpack(...) \
            RACTupleUnpack_(__VA_ARGS__)

    /// This and everything below is for internal use only.
    ///
    /// See RACTuplePack() and RACTupleUnpack() instead.
    #define RACTuplePack_(...) \
        ([RACTuplePack_class_name(__VA_ARGS__) tupleWithObjectsFromArray:@[ metamacro_foreach(RACTuplePack_object_or_ractuplenil,, __VA_ARGS__) ]])

    #define RACTuplePack_object_or_ractuplenil(INDEX, ARG) \
        (ARG) ?: RACTupleNil.tupleNil,

    /// 根据参数的个数 返回 元组的类名 1个对应 RACOneTuple ... 5个以上 返回RACTuple 最多20个
    #define RACTuplePack_class_name(...) \
            metamacro_at(20, __VA_ARGS__, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACTuple, RACFiveTuple, RACFourTuple, RACThreeTuple, RACTwoTuple, RACOneTuple)

    #define RACTupleUnpack_(...) \
        metamacro_foreach(RACTupleUnpack_decl,, __VA_ARGS__) \
        \
        int RACTupleUnpack_state = 0; \
        \
        RACTupleUnpack_after: \
            ; \
            metamacro_foreach(RACTupleUnpack_assign,, __VA_ARGS__) \
            if (RACTupleUnpack_state != 0) RACTupleUnpack_state = 2; \
            \
            while (RACTupleUnpack_state != 2) \
                if (RACTupleUnpack_state == 1) { \
                    goto RACTupleUnpack_after; \
                } else \
                    for (; RACTupleUnpack_state != 1; RACTupleUnpack_state = 1) \
                        [RACTupleUnpackingTrampoline trampoline][ @[ metamacro_foreach(RACTupleUnpack_value,, __VA_ARGS__) ] ]

    #define RACTupleUnpack_state metamacro_concat(RACTupleUnpack_state, __LINE__)
    #define RACTupleUnpack_after metamacro_concat(RACTupleUnpack_after, __LINE__)
    #define RACTupleUnpack_loop metamacro_concat(RACTupleUnpack_loop, __LINE__)

    #define RACTupleUnpack_decl_name(INDEX) \
        metamacro_concat(metamacro_concat(RACTupleUnpack, __LINE__), metamacro_concat(_var, INDEX))

    #define RACTupleUnpack_decl(INDEX, ARG) \
        __strong id RACTupleUnpack_decl_name(INDEX);

    #define RACTupleUnpack_assign(INDEX, ARG) \
        __strong ARG = RACTupleUnpack_decl_name(INDEX);

    #define RACTupleUnpack_value(INDEX, ARG) \
        [NSValue valueWithPointer:&RACTupleUnpack_decl_name(INDEX)],

    NS_ASSUME_NONNULL_BEGIN
    // 空元组
    @interface RACTupleNil : NSObject <NSCopying, NSCoding>
    /// A singleton instance.
    + (RACTupleNil *)tupleNil;
    @end

    @interface RACTuple : NSObject <NSCoding, NSCopying, NSFastEnumeration>

    @property (nonatomic, readonly) NSUInteger count;

    @property (nonatomic, readonly, nullable) id first;
    @property (nonatomic, readonly, nullable) id second;
    @property (nonatomic, readonly, nullable) id third;
    @property (nonatomic, readonly, nullable) id fourth;
    @property (nonatomic, readonly, nullable) id fifth;
    @property (nonatomic, readonly, nullable) id last;

     + (instancetype)tupleWithObjectsFromArray:(NSArray *)array { } //initWithBackingArray
    // 如果convert 为NO initWithBackingArray else 遍历array 如果object == NSNull.null 转换为RACTupleNil.tuplenil 最后initWithBackingArray
    + (instancetype)tupleWithObjectsFromArray:(NSArray *)array convertNullsToNils:(BOOL)convert {}
    // 遍历va_args 如果 != nil count++  如果count == 0 返回self init else  再遍历 加入数组中 用这个数组初始化实例、
    + (instancetype)tupleWithObjects:(id)object, ... {}
    // object == RACTupleNil.tupleNil ? nil : object
    - (id)objectAtIndex:(NSUInteger)index {}

    // 遍历 backingArray [newArray addObject:(object == RACTupleNil.tupleNil ? NSNull.null : object)];
    - (NSArray *)allObjects {    }
    // arrayByAddingObject生成一个新的array 再tupleWithObjectsFromArray生成一个新的对象 返回
    - (instancetype)tupleByAddingObject:(id)obj {  }

    @end

    @interface RACTuple (RACSequenceAdditions)

    @property (nonatomic, copy, readonly) RACSequence *rac_sequence;

    @end

    @interface RACTuple (ObjectSubscripting)
    - (nullable id)objectAtIndexedSubscript:(NSUInteger)idx;
    @end

    /// A tuple with exactly one generic value.
    @interface RACOneTuple<__covariant First> : RACTuple
    @property (nonatomic, readonly, nullable) First first;

    - (instancetype)init { return [self initWithBackingArray:@[ RACTupleNil.tupleNil ]]; }
    // 判断backingarray的count 是否等于1 等于1 super initWithArray:
    - (instancetype)initWithBackingArray:(NSArray *)backingArray {}
    // backingarray arrayByaddingObject 然后 RACTwoTuple tupleWithObjectsFromArray: newArray
    - (RACTwoTuple *)tupleByAddingObject:(id)obj { }

    + (instancetype)pack:(id)first { return [self tupleWithObjectsFromArray:@[ first ?: RACTupleNil.tupleNil,]];}
    // 和父类实现一样
    - (BOOL)isEqual:(RACTuple *)object {    }

    @end

    @interface RACTwoTuple<__covariant First, __covariant Second> : RACTuple
    + (instancetype)tupleWithObjects:(id)object, ... __attribute((unavailable("Use pack:: instead.")));
    - (RACThreeTuple<First, Second, id> *)tupleByAddingObject:(nullable id)obj;
    + (RACTwoTuple<First, Second> *)pack:(nullable First)first :(nullable Second)second;

    @property (nonatomic, readonly, nullable) First first;
    @property (nonatomic, readonly, nullable) Second second;

    @end

    @interface RACThreeTuple<__covariant First, __covariant Second, __covariant Third> : RACTuple
    + (instancetype)tupleWithObjects:(id)object, ... __attribute((unavailable("Use pack::: instead.")));
    - (RACFourTuple<First, Second, Third, id> *)tupleByAddingObject:(nullable id)obj;
    + (instancetype)pack:(nullable First)first :(nullable Second)second :(nullable Third)third;
    @property (nonatomic, readonly, nullable) First first;
    @property (nonatomic, readonly, nullable) Second second;
    @property (nonatomic, readonly, nullable) Third third;

    @end

    @interface RACFourTuple<__covariant First, __covariant Second, __covariant Third, __covariant Fourth> : RACTuple
    + (instancetype)tupleWithObjects:(id)object, ... __attribute((unavailable("Use pack:::: instead.")));
    - (RACFiveTuple<First, Second, Third, Fourth, id> *)tupleByAddingObject:(nullable id)obj;
    + (instancetype)pack:(nullable First)first :(nullable Second)second :(nullable Third)third :(nullable Fourth)fourth;
    @property (nonatomic, readonly, nullable) First first;
    @property (nonatomic, readonly, nullable) Second second;
    @property (nonatomic, readonly, nullable) Third third;
    @property (nonatomic, readonly, nullable) Fourth fourth;

    @end

    @interface RACFiveTuple<__covariant First, __covariant Second, __covariant Third, __covariant Fourth, __covariant Fifth> : RACTuple
    + (instancetype)tupleWithObjects:(id)object, ... __attribute((unavailable("Use pack::::: instead.")));
    + (instancetype)pack:(nullable First)first :(nullable Second)second :(nullable Third)third :(nullable Fourth)fourth :(nullable Fifth)fifth;

    @property (nonatomic, readonly, nullable) First first;
    @property (nonatomic, readonly, nullable) Second second;
    @property (nonatomic, readonly, nullable) Third third;
    @property (nonatomic, readonly, nullable) Fourth fourth;
    @property (nonatomic, readonly, nullable) Fifth fifth;

    @end

    @interface RACTupleUnpackingTrampoline : NSObject
    // signleton
    + (instancetype)trampoline;
    // 用NSValue 遍历variables 获取每个value的pointerValue 把指针指向tuple对应的下标的值
    - (void)setObject:(nullable RACTuple *)tuple forKeyedSubscript:(NSArray *)variables;
    @end


    .m 
    @interface RACTuple ()
    // 
    - (instancetype)initWithBackingArray:(NSArray *)backingArray NS_DESIGNATED_INITIALIZER;
    @property (nonatomic, readonly) NSArray *backingArray;
    @end

    @implementation RACTuple
    - (instancetype)init {}         //initWithBackingArray:@[]
    - (instancetype)initWithBackingArray:(NSArray *)backingArray {}     //_backingArray = backingArray
    - (NSString *)description {}  // self.class, self, self.allObjects]
    - (BOOL)isEqual:(RACTuple *)object {}       // 首先判断 == 再判断类 最后判断 backingArray isequal
    - (NSUInteger)hash {}               // self.backingArray.hash;
    // 直接用backingArray 调用同名方法
    - (NSUInteger)countByEnumeratingWithState:(NSFastEnumerationState *)state objects:(id __unsafe_unretained [])buffer count:(NSUInteger)len {}
    - (instancetype)copyWithZone:(NSZone *)zone {    } // return self;

    #pragma mark NSCoding

    - (instancetype)initWithCoder:(NSCoder *)coder { }

    - (void)encodeWithCoder:(NSCoder *)coder {
        if (self.backingArray != nil) [coder encodeObject:self.backingArray forKey:@keypath(self.backingArray)];
    }

    #pragma mark API

    - (NSUInteger)count { return self.backingArray.count; }
    - (id)first { return self[0]; }
    - (id)second { }
    - (id)third { }
    - (id)fourth { }
    - (id)fifth { }
    - (id)last {return self[self.count - 1];}

    @end

    @implementation RACTuple (RACSequenceAdditions)

    - (RACSequence *)rac_sequence { return [RACTupleSequence sequenceWithTupleBackingArray:self.backingArray offset:0]; }

    @end

    @implementation RACTuple (ObjectSubscripting)

    - (id)objectAtIndexedSubscript:(NSUInteger)idx { return [self objectAtIndex:idx]; }

    @end

    @implementation RACOneTuple
    @end

    @implementation RACTwoTuple
    @end

    @implementation RACThreeTuple
    @end

    @implementation RACFourTuple
    @end

    @implementation RACFiveTuple
    @end

    @implementation RACTupleUnpackingTrampoline
    @end

16. RACImmediateScheduler

    // [super initWithName:@"org.reactivecocoa.ReactiveObjC.RACScheduler.immediateScheduler"];
    - (instancetype)init {}

    #pragma mark RACScheduler
    // 直接执行block return nil
    - (RACDisposable *)schedule:(void (^)(void))block {}
    // date 和 block 非空 nsthread sleepuntilDate 执行block return nil
    - (RACDisposable *)after:(NSDate *)date schedule:(void (^)(void))block {}
    // 不支持此方法
    - (RACDisposable *)after:(NSDate *)date repeatingEvery:(NSTimeInterval)interval withLeeway:(NSTimeInterval)leeway schedule:(void (^)(void))block {}
    //
    - (RACDisposable *)scheduleRecursiveBlock:(RACSchedulerRecursiveBlock)recursiveBlock {}

17. RACSubscriptionScheduler
    // 
    @property (nonatomic, strong, readonly) RACScheduler *backgroundScheduler;
    // super initWithName: _backgroundScheduler = [RACScheduler scheduler]
    - (instancetype)init {}

    #pragma mark RACScheduler
    // 如果 currentScheduler 为空 _backgroundScheduler schedule:block 否则直接执行block retrun nil;
    - (RACDisposable *)schedule:(void (^)(void))block {}
    // 如果currentScheduler 非空 就用currentScheduler 否则backgroundScheduler after schedule
    - (RACDisposable *)after:(NSDate *)date schedule:(void (^)(void))block { }
    // 同上
    - (RACDisposable *)after:(NSDate *)date repeatingEvery:(NSTimeInterval)interval withLeeway:(NSTimeInterval)leeway schedule:(void (^)(void))block { }

18. RACTargetQueueScheduler
    // name 中把name 和queue的label 拼接在一起 创建一个新的串行队列 新的queue set_target_queue(targetQuque)
    - (instancetype)initWithName:(nullable NSString *)name targetQueue:(dispatch_queue_t)targetQueue;

19. RACSequence  <__covariant ValueType> : RACStream <NSCoding, NSCopying, NSFastEnumeration>
    // 抽象方法 虽然是属性 必须子类实现 自身是没有实现的
    @property (nonatomic, strong, readonly, nullable) ValueType head;
    // 同上
    @property (nonatomic, strong, readonly, nullable) RACSequence<ValueType> *tail;

    /// for 循环self 添加到 array中 return array
    @property (nonatomic, copy, readonly) NSArray<ValueType> *array;

    /// 新建一个RACSequenceEnumerator  sequence = self return 
    @property (nonatomic, copy, readonly) NSEnumerator<ValueType> *objectEnumerator;

    ///[RACEagerSequence sequenceWithArray:self.array offset:0];
    @property (nonatomic, copy, readonly) RACSequence<ValueType> *eagerSequence;

    /// return self
    @property (nonatomic, copy, readonly) RACSequence<ValueType> *lazySequence;

    /// [[self signalWithScheduler:[RACScheduler scheduler]] setNameWithFormat:@"[%@] -signal", self.name];
    - (RACSignal<ValueType> *)signal;

    /*
    return RACSignal createSignal:  ] setNameWithFormat:@"[%@] -signalWithScheduler: %@", self.name, scheduler
    在block 内 sequence = self; return scheduler schedulerRecursiveBlock: block内
    if reschedule.head 为nil subscriver sendCompleted return  subscriber sendNext:sequence.head sequence = sequence.tail reschedule()
    */
    - (RACSignal<ValueType> *)signalWithScheduler:(RACScheduler *)scheduler;

    /* 如果self.head 为 nil return start 
    reset = RACSequence sequenceWithHeadBlock:  return reduce(self.head, rest)
    在block内 if self.tail return self.tail foldLeftWithStart:start reduce:reduce else return start
     */
    - (id)foldLeftWithStart:(nullable id)start reduce:(id _Nullable (^)(id _Nullable accumulator, ValueType _Nullable value))reduce;

    /* 如果self.head 为 nil return start 
    遍历self start = reduce(start, value) return start
     */
    - (id)foldRightWithStart:(nullable id)start reduce:(id _Nullable (^)(id _Nullable first, RACSequence *rest))reduce;

    /// [self objectPassingTest:block] != nil;
    - (BOOL)any:(BOOL (^)(ValueType _Nullable value))block;

    ///  调用 foldLeftWithStart:@YES reduce return result.boolValue  在block内 @(accumulator && block(value))
    - (BOOL)all:(BOOL (^)(ValueType _Nullable value))block;

    ///  [self filter:block].head;
    - (nullable ValueType)objectPassingTest:(BOOL (^)(ValueType _Nullable value))block;
    // 调用 RACDynamicSequence的同名方法
    + (RACSequence<ValueType> *)sequenceWithHeadBlock:(ValueType _Nullable (^)(void))headBlock tailBlock:(nullable RACSequence<ValueType> *(^)(void))tailBlock;

    @end

    @interface RACSequence<__covariant ValueType> (RACStream)

    /// RACUnarySequence return:value
    + (RACSequence<ValueType> *)return:(nullable ValueType)value;

    /// RACEmptySequence.empty
    + (RACSequence<ValueType> *)empty;

    typedef RACSequence * _Nullable (^RACSequenceBindBlock)(ValueType _Nullable value, BOOL *stop);
    // 运行block 获取返回的 RACSquenceBindBlock 然后调用私有的 bind:passingThroughValuesFromSequence:nil 
    - (RACSequence *)bind:(RACSequenceBindBlock (^)(void))block;

    /// [[[RACArraySequence sequenceWithArray:@[ self, sequence ] offset:0] flatten] setNameWithFormat:@"[%@] -concat: %@", self.name, sequence];
    - (RACSequence *)concat:(RACSequence *)sequence;

    /*
    RACSequence  sequenceWithHeadBlock:tailBlock:   setNameWithFormat
    headBlock 如果head或者tail == nil return nil return RACTuplePack(self.head, sequence.head)
    tailBlock 如果self.tail == nil 或 self.tail isequal:RACSequence empty return nil  然后同样比较 sequence.tail 
    return self.tail zipWith:sequence.tail
    */
    - (RACSequence<RACTuple *> *)zipWith:(RACSequence *)sequence;

    @end

    /// Redeclarations of operations built on the RACStream primitives with more
    /// precise type information.
    @interface RACSequence<__covariant ValueType> (RACStreamOperations)

    - (RACSequence *)flattenMap:(__kindof RACSequence * _Nullable (^)(ValueType _Nullable value))block;
    - (RACSequence *)flatten;
    - (RACSequence *)map:(id _Nullable (^)(ValueType _Nullable value))block;
    - (RACSequence *)mapReplace:(nullable id)object;
    - (RACSequence<ValueType> *)filter:(BOOL (^)(id _Nullable value))block;
    - (RACSequence *)ignore:(nullable ValueType)value;
    - (RACSequence *)reduceEach:(RACReduceBlock)reduceBlock;
    - (RACSequence<ValueType> *)startWith:(nullable ValueType)value;
    - (RACSequence<ValueType> *)skip:(NSUInteger)skipCount;
    - (RACSequence<ValueType> *)take:(NSUInteger)count;
    + (RACSequence<RACTuple *> *)zip:(id<NSFastEnumeration>)sequence;
    + (RACSequence<ValueType> *)zip:(id<NSFastEnumeration>)sequences reduce:(RACReduceBlock)reduceBlock;
    /// Returns a sequence obtained by concatenating `sequences` in order.
    + (RACSequence<ValueType> *)concat:(id<NSFastEnumeration>)sequences;
    - (RACSequence *)scanWithStart:(nullable id)startingValue reduce:(id _Nullable (^)(id _Nullable running, ValueType _Nullable next))reduceBlock;
    - (RACSequence *)scanWithStart:(nullable id)startingValue reduceWithIndex:(id _Nullable (^)(id _Nullable running, ValueType _Nullable next, NSUInteger index))reduceBlock;
    - (RACSequence *)combinePreviousWithStart:(nullable ValueType)start reduce:(id _Nullable (^)(ValueType _Nullable previous, ValueType _Nullable current))reduceBlock;
    - (RACSequence<ValueType> *)takeUntilBlock:(BOOL (^)(ValueType _Nullable x))predicate;
    - (RACSequence<ValueType> *)takeWhileBlock:(BOOL (^)(ValueType _Nullable x))predicate;
    - (RACSequence<ValueType> *)skipUntilBlock:(BOOL (^)(ValueType _Nullable x))predicate;
    - (RACSequence<ValueType> *)skipWhileBlock:(BOOL (^)(ValueType _Nullable x))predicate;
    - (RACSequence<ValueType> *)distinctUntilChanged;
    @end

    .m

    // 内部枚举sequence的 枚举器
    @interface RACSequenceEnumerator : NSEnumerator
    @property (nonatomic, strong) RACSequence *sequence;
    @end
    @implementation RACSequenceEnumerator
    // id object  同步锁  object = self.sequence.head self.sequence = self.sequence.tail return object;
    - (id)nextObject { }

    @end

    @interface RACSequence ()
    /* 
    RACSequence valuseq = self  current = passthroughSequence BOOL stop = NO;
    通过RACDynamicSequence sequecnWithlazyDependency:headBlock:tailBlock: 新建一个sequence sequenc.name = self.name 返回
    三个block while 循环 直到 curren.head == nil 在循环内 如果stop return nil  id value = self.head 如果value 为nil stop = yes return nil
    current = bindblock(value, &stop) 如果current == nil stop = yes return nil valueSeq = valueSeq.tail
    headBlock: return current.head
    tailBlock: 如果stop = yes return nil else return valueSeq bind:passThroughValuesFromSequence:current.tail
    */ 
    - (RACSequence *)bind:(RACSequenceBindBlock)block passingThroughValuesFromSequence:(RACSequence *)current;

    - (id)copyWithZone:(NSZone *)zone {
        return self;
    }

    #pragma mark NSCoding

    - (Class)classForCoder { }
    - (id)initWithCoder:(NSCoder *)coder {}

    - (void)encodeWithCoder:(NSCoder *)coder {
        [coder encodeObject:self.array forKey:@"array"];
    }
    - (NSUInteger)countByEnumeratingWithState:(NSFastEnumerationState *)state objects:(__unsafe_unretained id *)stackbuf count:(NSUInteger)len { }

    #pragma mark NSObject
    - (NSUInteger)hash { }
    - (BOOL)isEqual:(RACSequence *)seq {}     
    @end

20. RACEmptySequence 
    
    // singleton
    + (RACEmptySequence *)empty;
    // .m
    - (id)head { return nil; }
    - (RACSequence *)tail { return nil; }
    // return passthroughSequence ?: self;
    - (RACSequence *)bind:(RACStreamBindBlock)bindBlock passingThroughValuesFromSequence:(RACSequence *)passthroughSequence {}
    - (Class)classForCoder { return self.class; }
    - (instancetype)initWithCoder:(NSCoder *)coder { return self.class.empty; }
    - (void)encodeWithCoder:(NSCoder *)coder {}
    - (NSString *)description { return [NSString stringWithFormat:@"<%@: %p> { name = %@ }", self.class, self, self.name]; }
    - (NSUInteger)hash { return (NSUInteger)(__bridge void *)self; }
    - (BOOL)isEqual:(RACSequence *)seq { return (self == seq); }

21. RACTupleSequence

    //  新建一个RACTupleSequence  一一赋值 如果offset == array count return empty
    + (RACSequence *)sequenceWithTupleBackingArray:(NSArray *)backingArray offset:(NSUInteger)offset; 

    // .m 
    @property (nonatomic, strong, readonly) NSArray *tupleBackingArray;
    @property (nonatomic, assign, readonly) NSUInteger offset;

    @implementation RACTupleSequence

    #pragma mark RACSequence
    // 返回tupleBackingArray[offset] 如果 == RACTupleNil.tupleNil NSNull.null 
    - (id)head { }
    // RACSequence sequenceWithTupleBackingArray:self.tupleBackingArray offset:self.offset + 1 返回
    - (RACSequence *)tail { }
    // 从offset 到最后一个 遍历 如果 当前Object == RACTupleNil.tupleNil 加入NSNull.null 否则加入object 返回MutableArray
    - (NSArray *)array { } 
    - (NSString *)description { }
    @end

22. RACDynamicSequence 
    // new RACDynamicSequence 三个block 一一赋值 然后_dependency = YES;
    + (RACSequence *)sequenceWithLazyDependency:(id (^)(void))dependencyBlock headBlock:(id (^)(id dependency))headBlock tailBlock:(RACSequence *(^)(id dependency))tailBlock;
    
    // .m
    #define DEALLOC_OVERFLOW_GUARD 100
    @interface RACDynamicSequence () {
        id _head;
        RACSequence *_tail;
        id _dependency;
    }

    // 
    @property (nonatomic, strong) id headBlock;
    //  
    @property (nonatomic, strong) id tailBlock;
    //  
    @property (nonatomic, assign) BOOL hasDependency;
    //  
    @property (nonatomic, strong) id (^dependencyBlock)(void);

    @end

    @implementation RACDynamicSequence
    // new RACDynamicSequence  seq  headBlock tailBlokc 赋值  然后 _hasDependency = NO
    + (RACSequence *)sequenceWithHeadBlock:(id (^)(void))headBlock tailBlock:(RACSequence<id> *(^)(void))tailBlock {}
    // static count 如果 count >= 100 count -= 100 然后 tail = _tail  放入自动释放池 _tail = nil
    - (void)dealloc {}
    /*
    同步锁 id uheadBlock = _headBlock 如果 为空 返回 _head 如果 _hadDependency 
    再判断 dependencyBlock 是否为空 不为空 _dependency = dependencyBlock 返回的结果 dependencyBlock 置nil _head = uheadBlock(_dependency)
    _hadDependency = NO 直接 _head = uheadBlock()    最后_headBlock = nil return _head 
    */ 
    - (id)head { }
    /* 过程同上 _tail = tailBlock  return _tail */
    - (RACSequence *)tail { }
    // 
    - (NSString *)description { }

    @end

23. RACEagerSequence

    // .m
    // sequenceWithArray:@[ value ] offset:0] setNameWithFormat:@"+return: %@", RACDescription(value) 返回一个新的RACArraySequence
    + (RACSequence *)return:(id)value { }
    /*
    block 转换为 RACStreamBindBlock 遍历self.array 调用 bindBlock 如果返回的为nil 停止循环没停止的话 遍历返回的RACArraySequence
    加入到MutableArray中 如果 bindBlock的第二个参数变为了YES 停止最外层的循环 
    最后返回 sequenceWithArray:mutableArray offset:0] setNameWithFormat:@"[%@] -bind:", self.name] 
    */
    - (RACSequence *)bind:(RACSequenceBindBlock (^)(void))block { }
    // 将sequence的array和自己的array合并 返回新的 RACArraySequence
    - (RACSequence *)concat:(RACSequence *)sequence { }
    - (RACSequence *)eagerSequence { return self }
    // 新的 RACArraySequence
    - (RACSequence *)lazySequence { }
    // 调用super 同名方法 在block 内 调用reduce(first, rest.eagerSequence)
    - (id)foldRightWithStart:(id)start reduce:(id (^)(id, RACSequence *rest))reduce { }

24. RACArraySequence

    // 新建一个RACArraySequence  一一赋值 如果offset == array count return empty
    + (RACSequence *)sequenceWithArray:(NSArray *)array offset:(NSUInteger)offset {}
    // .m
    // 将父类的array 申明废弃  然后 使用自身的backingArray
    @property (nonatomic, copy, readonly) NSArray *array __attribute__((deprecated));
    @property (nonatomic, copy, readonly) NSArray *backingArray;
    @property (nonatomic, assign, readonly) NSUInteger offset;

    @end

    @implementation RACArraySequence

    - (id)head { return self.backingArray[self.offset]; }
    // 用self.backingArray offset + 1 新建一个ArraySequence 返回
    - (RACSequence *)tail { }

    - (NSUInteger)countByEnumeratingWithState:(NSFastEnumerationState *)state objects:(__unsafe_unretained id[])stackbuf count:(NSUInteger)len { }

    #pragma clang diagnostic push
    #pragma clang diagnostic ignored "-Wdeprecated-implementations"
    // 返回 backingarray 从offset 到最后一个的新的array
    - (NSArray *)array { }
    #pragma clang diagnostic pop
    // super initWithCoder  _backingArray = [coder decodeObjectForKey:@"array"]; _offset = 0;
    - (instancetype)initWithCoder:(NSCoder *)coder {  }
    // super
    - (void)encodeWithCoder:(NSCoder *)coder { }
    - (NSString *)description { }

26. RACUnarySequence
    // new RACUnarySequence head = value
    + (RACUnarySequence *)return:(id)value;
    // .m 
    @interface RACUnarySequence ()

    @property (nonatomic, strong, readwrite) id head;
    @end

    @implementation RACUnarySequence 
    @synthesize head = _head;
    - (RACSequence *)tail { }
    // block 转换为RACStreamBindBlock 调用转换后的block  拿到新的RACSequence  如果新的为空返回 empty 
    - (RACSequence *)bind:(RACSequenceBindBlock (^)(void))block {}
    - (Class)classForCoder { return self.class; }
    - (instancetype)initWithCoder:(NSCoder *)coder { }
    - (void)encodeWithCoder:(NSCoder *)coder { }
    - (NSString *)description { }
    - (NSUInteger)hash { }
    - (BOOL)isEqual:(RACUnarySequence *)seq { }

    @end

27. RACUnit

    /// A singleton instance.
    + (RACUnit *)defaultUnit;

28. RACBehaviorSubject RACBehaviorSubject<ValueType> : RACSubject<ValueType>

/// Creates a new behavior subject with a default value. If it hasn't received
/// any values when it gets subscribed to, it sends the default value.
+ (instancetype)behaviorSubjectWithDefaultValue:(nullable ValueType)value;

@interface RACBehaviorSubject<ValueType> ()

// This property should only be used while synchronized on self.
@property (nonatomic, strong) ValueType currentValue;

@end

@implementation RACBehaviorSubject

#pragma mark Lifecycle

+ (instancetype)behaviorSubjectWithDefaultValue:(id)value {
	RACBehaviorSubject *subject = [self subject];
	subject.currentValue = value;
	return subject;
}

#pragma mark RACSignal

- (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {
	RACDisposable *subscriptionDisposable = [super subscribe:subscriber];

	RACDisposable *schedulingDisposable = [RACScheduler.subscriptionScheduler schedule:^{
		@synchronized (self) {
			[subscriber sendNext:self.currentValue];
		}
	}];
	
	return [RACDisposable disposableWithBlock:^{
		[subscriptionDisposable dispose];
		[schedulingDisposable dispose];
	}];
}

#pragma mark RACSubscriber

- (void)sendNext:(id)value {
	@synchronized (self) {
		self.currentValue = value;
		[super sendNext:value];
	}
}

29. RACBlockTrampoline


+ (id)invokeBlock:(id)block withArguments:(RACTuple *)arguments;

@interface RACBlockTrampoline ()
@property (nonatomic, readonly, copy) id block;
@end

@implementation RACBlockTrampoline

#pragma mark API

- (instancetype)initWithBlock:(id)block {
	self = [super init];

	_block = [block copy];

	return self;
}

+ (id)invokeBlock:(id)block withArguments:(RACTuple *)arguments {
	NSCParameterAssert(block != NULL);

	RACBlockTrampoline *trampoline = [(RACBlockTrampoline *)[self alloc] initWithBlock:block];
	return [trampoline invokeWithArguments:arguments];
}

- (id)invokeWithArguments:(RACTuple *)arguments {
	SEL selector = [self selectorForArgumentCount:arguments.count];
	NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:[self methodSignatureForSelector:selector]];
	invocation.selector = selector;
	invocation.target = self;

	for (NSUInteger i = 0; i < arguments.count; i++) {
		id arg = arguments[i];
		NSInteger argIndex = (NSInteger)(i + 2);
		[invocation setArgument:&arg atIndex:argIndex];
	}

	[invocation invoke];
	
	__unsafe_unretained id returnVal;
	[invocation getReturnValue:&returnVal];
	return returnVal;
}

- (SEL)selectorForArgumentCount:(NSUInteger)count {
	NSCParameterAssert(count > 0);

	switch (count) {
		case 0: return NULL;
		case 1: return @selector(performWith:);
		case 2: return @selector(performWith::);
		case 3: return @selector(performWith:::);
		case 4: return @selector(performWith::::);
		case 5: return @selector(performWith:::::);
		case 6: return @selector(performWith::::::);
		case 7: return @selector(performWith:::::::);
		case 8: return @selector(performWith::::::::);
		case 9: return @selector(performWith:::::::::);
		case 10: return @selector(performWith::::::::::);
		case 11: return @selector(performWith:::::::::::);
		case 12: return @selector(performWith::::::::::::);
		case 13: return @selector(performWith:::::::::::::);
		case 14: return @selector(performWith::::::::::::::);
		case 15: return @selector(performWith:::::::::::::::);
	}

	NSCAssert(NO, @"The argument count is too damn high! Only blocks of up to 15 arguments are currently supported.");
	return NULL;
}

- (id)performWith:(id)obj1 {
	id (^block)(id) = self.block;
	return block(obj1);
}

- (id)performWith:(id)obj1 :(id)obj2 {
	id (^block)(id, id) = self.block;
	return block(obj1, obj2);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 {
	id (^block)(id, id, id) = self.block;
	return block(obj1, obj2, obj3);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 {
	id (^block)(id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 {
	id (^block)(id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 :(id)obj6 {
	id (^block)(id, id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5, obj6);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 :(id)obj6 :(id)obj7 {
	id (^block)(id, id, id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5, obj6, obj7);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 :(id)obj6 :(id)obj7 :(id)obj8 {
	id (^block)(id, id, id, id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5, obj6, obj7, obj8);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 :(id)obj6 :(id)obj7 :(id)obj8 :(id)obj9 {
	id (^block)(id, id, id, id, id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5, obj6, obj7, obj8, obj9);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 :(id)obj6 :(id)obj7 :(id)obj8 :(id)obj9 :(id)obj10 {
	id (^block)(id, id, id, id, id, id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5, obj6, obj7, obj8, obj9, obj10);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 :(id)obj6 :(id)obj7 :(id)obj8 :(id)obj9 :(id)obj10 :(id)obj11 {
	id (^block)(id, id, id, id, id, id, id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5, obj6, obj7, obj8, obj9, obj10, obj11);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 :(id)obj6 :(id)obj7 :(id)obj8 :(id)obj9 :(id)obj10 :(id)obj11 :(id)obj12 {
	id (^block)(id, id, id, id, id, id, id, id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5, obj6, obj7, obj8, obj9, obj10, obj11, obj12);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 :(id)obj6 :(id)obj7 :(id)obj8 :(id)obj9 :(id)obj10 :(id)obj11 :(id)obj12 :(id)obj13 {
	id (^block)(id, id, id, id, id, id, id, id, id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5, obj6, obj7, obj8, obj9, obj10, obj11, obj12, obj13);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 :(id)obj6 :(id)obj7 :(id)obj8 :(id)obj9 :(id)obj10 :(id)obj11 :(id)obj12 :(id)obj13 :(id)obj14 {
	id (^block)(id, id, id, id, id, id, id, id, id, id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5, obj6, obj7, obj8, obj9, obj10, obj11, obj12, obj13, obj14);
}

- (id)performWith:(id)obj1 :(id)obj2 :(id)obj3 :(id)obj4 :(id)obj5 :(id)obj6 :(id)obj7 :(id)obj8 :(id)obj9 :(id)obj10 :(id)obj11 :(id)obj12 :(id)obj13 :(id)obj14 :(id)obj15 {
	id (^block)(id, id, id, id, id, id, id, id, id, id, id, id, id, id, id) = self.block;
	return block(obj1, obj2, obj3, obj4, obj5, obj6, obj7, obj8, obj9, obj10, obj11, obj12, obj13, obj14, obj15);
}

30. RACCommand

    /// The domain for errors originating within `RACCommand`.
extern NSErrorDomain const RACCommandErrorDomain;

typedef NS_ERROR_ENUM(RACCommandErrorDomain, RACCommandError) {
	/// -execute: was invoked while the command was disabled.
	RACCommandErrorNotEnabled = 1,
};

/// A `userInfo` key for an error, associated with the `RACCommand` that the
/// error originated from.
///
/// This is included only when the error code is `RACCommandErrorNotEnabled`.
extern NSString * const RACUnderlyingCommandErrorKey;

/// A command is a signal triggered in response to some action, typically
/// UI-related.
@interface RACCommand<__contravariant InputType, __covariant ValueType> : NSObject

/// A signal of the signals returned by successful invocations of -execute:
/// (i.e., while the receiver is `enabled`).
///
/// Errors will be automatically caught upon the inner signals, and sent upon
/// `errors` instead. If you _want_ to receive inner errors, use -execute: or
/// -[RACSignal materialize].
/// 
/// Only executions that begin _after_ subscription will be sent upon this
/// signal. All inner signals will arrive upon the main thread.
@property (nonatomic, strong, readonly) RACSignal<RACSignal<ValueType> *> *executionSignals;

/// A signal of whether this command is currently executing.
///
/// This will send YES whenever -execute: is invoked and the created signal has
/// not yet terminated. Once all executions have terminated, `executing` will
/// send NO.
///
/// This signal will send its current value upon subscription, and then all
/// future values on the main thread.
@property (nonatomic, strong, readonly) RACSignal<NSNumber *> *executing;

/// A signal of whether this command is able to execute.
///
/// This will send NO if:
///
///  - The command was created with an `enabledSignal`, and NO is sent upon that
///    signal, or
///  - `allowsConcurrentExecution` is NO and the command has started executing.
///
/// Once the above conditions are no longer met, the signal will send YES.
///
/// This signal will send its current value upon subscription, and then all
/// future values on the main thread.
@property (nonatomic, strong, readonly) RACSignal<NSNumber *> *enabled;

/// Forwards any errors that occur within signals returned by -execute:.
///
/// When an error occurs on a signal returned from -execute:, this signal will
/// send the associated NSError value as a `next` event (since an `error` event
/// would terminate the stream).
///
/// After subscription, this signal will send all future errors on the main
/// thread.
@property (nonatomic, strong, readonly) RACSignal<NSError *> *errors;

/// Whether the command allows multiple executions to proceed concurrently.
///
/// The default value for this property is NO.
@property (atomic, assign) BOOL allowsConcurrentExecution;

/// Invokes -initWithEnabled:signalBlock: with a nil `enabledSignal`.
- (instancetype)initWithSignalBlock:(RACSignal<ValueType> * (^)(InputType _Nullable input))signalBlock;

/// Initializes a command that is conditionally enabled.
///
/// This is the designated initializer for this class.
///
/// enabledSignal - A signal of BOOLs which indicate whether the command should
///                 be enabled. `enabled` will be based on the latest value sent
///                 from this signal. Before any values are sent, `enabled` will
///                 default to YES. This argument may be nil.
/// signalBlock   - A block which will map each input value (passed to -execute:)
///                 to a signal of work. The returned signal will be multicasted
///                 to a replay subject, sent on `executionSignals`, then
///                 subscribed to synchronously. Neither the block nor the
///                 returned signal may be nil.
- (instancetype)initWithEnabled:(nullable RACSignal<NSNumber *> *)enabledSignal signalBlock:(RACSignal<ValueType> * (^)(InputType _Nullable input))signalBlock;

/// If the receiver is enabled, this method will:
///
///  1. Invoke the `signalBlock` given at the time of initialization.
///  2. Multicast the returned signal to a RACReplaySubject.
///  3. Send the multicasted signal on `executionSignals`.
///  4. Subscribe (connect) to the original signal on the main thread.
///
/// input - The input value to pass to the receiver's `signalBlock`. This may be
///         nil.
///
/// Returns the multicasted signal, after subscription. If the receiver is not
/// enabled, returns a signal that will send an error with code
/// RACCommandErrorNotEnabled.
- (RACSignal<ValueType> *)execute:(nullable InputType)input;


NSErrorDomain const RACCommandErrorDomain = @"RACCommandErrorDomain";
NSString * const RACUnderlyingCommandErrorKey = @"RACUnderlyingCommandErrorKey";

@interface RACCommand () {
	// Atomic backing variable for `allowsConcurrentExecution`.
	volatile uint32_t _allowsConcurrentExecution;
}

/// A subject that sends added execution signals.
@property (nonatomic, strong, readonly) RACSubject *addedExecutionSignalsSubject;

/// A subject that sends the new value of `allowsConcurrentExecution` whenever it changes.
@property (nonatomic, strong, readonly) RACSubject *allowsConcurrentExecutionSubject;

// `enabled`, but without a hop to the main thread.
//
// Values from this signal may arrive on any thread.
@property (nonatomic, strong, readonly) RACSignal *immediateEnabled;

// The signal block that the receiver was initialized with.
@property (nonatomic, copy, readonly) RACSignal * (^signalBlock)(id input);

@end

@implementation RACCommand

#pragma mark Properties

- (BOOL)allowsConcurrentExecution {
	return _allowsConcurrentExecution != 0;
}

- (void)setAllowsConcurrentExecution:(BOOL)allowed {
	if (allowed) {
		OSAtomicOr32Barrier(1, &_allowsConcurrentExecution);
	} else {
		OSAtomicAnd32Barrier(0, &_allowsConcurrentExecution);
	}

	[self.allowsConcurrentExecutionSubject sendNext:@(_allowsConcurrentExecution)];
}

#pragma mark Lifecycle

- (instancetype)init {
	NSCAssert(NO, @"Use -initWithSignalBlock: instead");
	return nil;
}

- (instancetype)initWithSignalBlock:(RACSignal<id> * (^)(id input))signalBlock {
	return [self initWithEnabled:nil signalBlock:signalBlock];
}

- (void)dealloc {
	[_addedExecutionSignalsSubject sendCompleted];
	[_allowsConcurrentExecutionSubject sendCompleted];
}

- (instancetype)initWithEnabled:(RACSignal *)enabledSignal signalBlock:(RACSignal<id> * (^)(id input))signalBlock {
	NSCParameterAssert(signalBlock != nil);

	self = [super init];

	_addedExecutionSignalsSubject = [RACSubject new];
	_allowsConcurrentExecutionSubject = [RACSubject new];
	_signalBlock = [signalBlock copy];

	_executionSignals = [[[self.addedExecutionSignalsSubject
		map:^(RACSignal *signal) {
			return [signal catchTo:[RACSignal empty]];
		}]
		deliverOn:RACScheduler.mainThreadScheduler]
		setNameWithFormat:@"%@ -executionSignals", self];
	
	// `errors` needs to be multicasted so that it picks up all
	// `activeExecutionSignals` that are added.
	//
	// In other words, if someone subscribes to `errors` _after_ an execution
	// has started, it should still receive any error from that execution.
	RACMulticastConnection *errorsConnection = [[[self.addedExecutionSignalsSubject
		flattenMap:^(RACSignal *signal) {
			return [[signal
				ignoreValues]
				catch:^(NSError *error) {
					return [RACSignal return:error];
				}];
		}]
		deliverOn:RACScheduler.mainThreadScheduler]
		publish];
	
	_errors = [errorsConnection.signal setNameWithFormat:@"%@ -errors", self];
	[errorsConnection connect];

	RACSignal *immediateExecuting = [[[[self.addedExecutionSignalsSubject
		flattenMap:^(RACSignal *signal) {
			return [[[signal
				catchTo:[RACSignal empty]]
				then:^{
					return [RACSignal return:@-1];
				}]
				startWith:@1];
		}]
		scanWithStart:@0 reduce:^(NSNumber *running, NSNumber *next) {
			return @(running.integerValue + next.integerValue);
		}]
		map:^(NSNumber *count) {
			return @(count.integerValue > 0);
		}]
		startWith:@NO];

	_executing = [[[[[immediateExecuting
		deliverOn:RACScheduler.mainThreadScheduler]
		// This is useful before the first value arrives on the main thread.
		startWith:@NO]
		distinctUntilChanged]
		replayLast]
		setNameWithFormat:@"%@ -executing", self];
	
	RACSignal *moreExecutionsAllowed = [RACSignal
		if:[self.allowsConcurrentExecutionSubject startWith:@NO]
		then:[RACSignal return:@YES]
		else:[immediateExecuting not]];
	
	if (enabledSignal == nil) {
		enabledSignal = [RACSignal return:@YES];
	} else {
		enabledSignal = [enabledSignal startWith:@YES];
	}
	
	_immediateEnabled = [[[[RACSignal
		combineLatest:@[ enabledSignal, moreExecutionsAllowed ]]
		and]
		takeUntil:self.rac_willDeallocSignal]
		replayLast];
	
	_enabled = [[[[[self.immediateEnabled
		take:1]
		concat:[[self.immediateEnabled skip:1] deliverOn:RACScheduler.mainThreadScheduler]]
		distinctUntilChanged]
		replayLast]
		setNameWithFormat:@"%@ -enabled", self];

	return self;
}

#pragma mark Execution

- (RACSignal *)execute:(id)input {
	// `immediateEnabled` is guaranteed to send a value upon subscription, so
	// -first is acceptable here.
	BOOL enabled = [[self.immediateEnabled first] boolValue];
	if (!enabled) {
		NSError *error = [NSError errorWithDomain:RACCommandErrorDomain code:RACCommandErrorNotEnabled userInfo:@{
			NSLocalizedDescriptionKey: NSLocalizedString(@"The command is disabled and cannot be executed", nil),
			RACUnderlyingCommandErrorKey: self
		}];

		return [RACSignal error:error];
	}

	RACSignal *signal = self.signalBlock(input);
	NSCAssert(signal != nil, @"nil signal returned from signal block for value: %@", input);

	// We subscribe to the signal on the main thread so that it occurs _after_
	// -addActiveExecutionSignal: completes below.
	//
	// This means that `executing` and `enabled` will send updated values before
	// the signal actually starts performing work.
	RACMulticastConnection *connection = [[signal
		subscribeOn:RACScheduler.mainThreadScheduler]
		multicast:[RACReplaySubject subject]];
	
	[self.addedExecutionSignalsSubject sendNext:connection.signal];

	[connection connect];
	return [connection.signal setNameWithFormat:@"%@ -execute: %@", self, RACDescription(input)];
}

31. RACDelegateProxy

    @property (nonatomic, unsafe_unretained) id rac_proxiedDelegate;
    // class_addProtocol 声明自己实现了这个协议 _protocol = protocol
    - (instancetype)initWithProtocol:(Protocol *)protocol;

    - (RACSignal *)signalForSelector:(SEL)selector;
    // .m
    {
        Protocol *_protocol;
    }
    //  rac_signalForSelector:rac_signalForSelector:
    - (RACSignal *)signalForSelector:(SEL)selector { }

    - (BOOL)isProxy {   return YES;    }

    - (void)forwardInvocation:(NSInvocation *)invocation { }

    - (NSMethodSignature *)methodSignatureForSelector:(SEL)selector {  }

    - (BOOL)respondsToSelector:(SEL)selector { }

32. RACEvent<__covariant ValueType> : NSObject <NSCopying>

/// Returns a singleton RACEvent representing the `completed` event.
+ (RACEvent<ValueType> *)completedEvent;

/// Returns a new event of type RACEventTypeError, containing the given error.
+ (RACEvent<ValueType> *)eventWithError:(nullable NSError *)error;

/// Returns a new event of type RACEventTypeNext, containing the given value.
+ (RACEvent<ValueType> *)eventWithValue:(nullable ValueType)value;

/// The type of event represented by the receiver.
@property (nonatomic, assign, readonly) RACEventType eventType;

/// Returns whether the receiver is of type RACEventTypeCompleted or
/// RACEventTypeError.
@property (nonatomic, getter = isFinished, assign, readonly) BOOL finished;

/// The error associated with an event of type RACEventTypeError. This will be
/// nil for all other event types.
@property (nonatomic, strong, readonly, nullable) NSError *error;

/// The value associated with an event of type RACEventTypeNext. This will be
/// nil for all other event types.
@property (nonatomic, strong, readonly, nullable) ValueType value;


// An object associated with this event. This will be used for the error and
// value properties.
@property (nonatomic, strong, readonly) id object;

// Initializes the receiver with the given type and object.
- (instancetype)initWithEventType:(RACEventType)type object:(id)object;

@end

@implementation RACEvent

#pragma mark Properties

- (BOOL)isFinished {
	return self.eventType == RACEventTypeCompleted || self.eventType == RACEventTypeError;
}

- (NSError *)error {
	return (self.eventType == RACEventTypeError ? self.object : nil);
}

- (id)value {
	return (self.eventType == RACEventTypeNext ? self.object : nil);
}

#pragma mark Lifecycle

+ (instancetype)completedEvent {
	static dispatch_once_t pred;
	static id singleton;

	dispatch_once(&pred, ^{
		singleton = [[self alloc] initWithEventType:RACEventTypeCompleted object:nil];
	});

	return singleton;
}

+ (instancetype)eventWithError:(NSError *)error {
	return [[self alloc] initWithEventType:RACEventTypeError object:error];
}

+ (instancetype)eventWithValue:(id)value {
	return [[self alloc] initWithEventType:RACEventTypeNext object:value];
}

- (instancetype)initWithEventType:(RACEventType)type object:(id)object {
	self = [super init];

	_eventType = type;
	_object = object;

	return self;
}

#pragma mark NSCopying

- (id)copyWithZone:(NSZone *)zone {
	return self;
}

#pragma mark NSObject

- (NSString *)description {
	NSString *eventDescription = nil;

	switch (self.eventType) {
		case RACEventTypeCompleted:
			eventDescription = @"completed";
			break;

		case RACEventTypeError:
			eventDescription = [NSString stringWithFormat:@"error = %@", self.object];
			break;

		case RACEventTypeNext:
			eventDescription = [NSString stringWithFormat:@"next = %@", self.object];
			break;

		default:
			NSCAssert(NO, @"Unrecognized event type: %i", (int)self.eventType);
	}

	return [NSString stringWithFormat:@"<%@: %p>{ %@ }", self.class, self, eventDescription];
}

- (NSUInteger)hash {
	return self.eventType ^ [self.object hash];
}

- (BOOL)isEqual:(id)event {
	if (event == self) return YES;
	if (![event isKindOfClass:RACEvent.class]) return NO;
	if (self.eventType != [event eventType]) return NO;

	// Catches the nil case too.
	return self.object == [event object] || [self.object isEqual:[event object]];
}


33. RACGroupedSignal : RACSubject


/// The key shared by the group.
@property (nonatomic, readonly, copy) id<NSCopying> key;

+ (instancetype)signalWithKey:(id<NSCopying>)key;
@interface RACGroupedSignal ()
@property (nonatomic, copy) id<NSCopying> key;
@end

@implementation RACGroupedSignal

#pragma mark API

+ (instancetype)signalWithKey:(id<NSCopying>)key {
	RACGroupedSignal *subject = [self subject];
	subject.key = key;
	return subject;
}


34. RACIndexSetSequence : RACSequence

+ (RACSequence *)sequenceWithIndexSet:(NSIndexSet *)indexSet;

@interface RACIndexSetSequence ()

// A buffer holding the `NSUInteger` values to enumerate over.
//
// This is mostly used for memory management. Most access should go through
// `indexes` instead.
@property (nonatomic, strong, readonly) NSData *data;

// The indexes that this sequence should enumerate.
@property (nonatomic, readonly) const NSUInteger *indexes;

// The number of indexes to enumerate.
@property (nonatomic, readonly) NSUInteger count;

@end

@implementation RACIndexSetSequence

#pragma mark Lifecycle

+ (RACSequence *)sequenceWithIndexSet:(NSIndexSet *)indexSet {
	NSUInteger count = indexSet.count;
	
	if (count == 0) return self.empty;
	
	NSUInteger sizeInBytes = sizeof(NSUInteger) * count;

	NSMutableData *data = [[NSMutableData alloc] initWithCapacity:sizeInBytes];
	[indexSet getIndexes:data.mutableBytes maxCount:count inIndexRange:NULL];

	RACIndexSetSequence *seq = [[self alloc] init];
	seq->_data = data;
	seq->_indexes = data.bytes;
	seq->_count = count;
	return seq;
}

+ (instancetype)sequenceWithIndexSetSequence:(RACIndexSetSequence *)indexSetSequence offset:(NSUInteger)offset {
	NSCParameterAssert(offset < indexSetSequence.count);

	RACIndexSetSequence *seq = [[self alloc] init];
	seq->_data = indexSetSequence.data;
	seq->_indexes = indexSetSequence.indexes + offset;
	seq->_count = indexSetSequence.count - offset;
	return seq;
}

#pragma mark RACSequence

- (id)head {
	return @(self.indexes[0]);
}

- (RACSequence *)tail {
	if (self.count <= 1) return [RACSequence empty];

	return [self.class sequenceWithIndexSetSequence:self offset:1];
}

#pragma mark NSFastEnumeration

- (NSUInteger)countByEnumeratingWithState:(NSFastEnumerationState *)state objects:(__unsafe_unretained id[])stackbuf count:(NSUInteger)len {
	NSCParameterAssert(len > 0);

	if (state->state >= self.count) {
		// Enumeration has completed.
		return 0;
	}
	
	if (state->state == 0) {
		// Enumeration begun, mark the mutation flag.
		state->mutationsPtr = state->extra;
	}
	
	state->itemsPtr = stackbuf;
	
	unsigned long index = 0;
	while (index < MIN(self.count - state->state, len)) {
		stackbuf[index] = @(self.indexes[index + state->state]);
		++index;
	}
	
	state->state += index;
	return index;
}

#pragma mark NSObject

- (NSString *)description {
	NSMutableString *indexesStr = [NSMutableString string];

	for (unsigned int i = 0; i < self.count; ++i) {
		if (i > 0) [indexesStr appendString:@", "];

		[indexesStr appendFormat:@"%lu", (unsigned long)self.indexes[i]];
	}

	return [NSString stringWithFormat:@"<%@: %p>{ name = %@, indexes = %@ }", self.class, self, self.name, indexesStr];
}

@end


35. #define RACChannelTo(TARGET, ...) \
    metamacro_if_eq(1, metamacro_argcount(__VA_ARGS__)) \
        (RACChannelTo_(TARGET, __VA_ARGS__, nil)) \
        (RACChannelTo_(TARGET, __VA_ARGS__))

/// Do not use this directly. Use the RACChannelTo macro above.
#define RACChannelTo_(TARGET, KEYPATH, NILVALUE) \
    [[RACKVOChannel alloc] initWithTarget:(TARGET) keyPath:@keypath(TARGET, KEYPATH) nilValue:(NILVALUE)][@keypath(RACKVOChannel.new, followingTerminal)]

NS_ASSUME_NONNULL_BEGIN

/// A RACChannel that observes a KVO-compliant key path for changes.
@interface RACKVOChannel<ValueType> : RACChannel<ValueType>

/// Initializes a channel that will observe the given object and key path.
///
/// The current value of the key path, and future KVO notifications for the given
/// key path, will be sent to subscribers of the channel's `followingTerminal`.
/// Values sent to the `followingTerminal` will be set at the given key path using
/// key-value coding.
///
/// When the target object deallocates, the channel will complete. Signal errors
/// are considered undefined behavior.
///
/// This is the designated initializer for this class.
///
/// target   - The object to bind to.
/// keyPath  - The key path to observe and set the value of.
/// nilValue - The value to set at the key path whenever a `nil` value is
///            received. This may be nil when connecting to object properties, but
///            an NSValue should be used for primitive properties, to avoid an
///            exception if `nil` is received (which might occur if an intermediate
///            object is set to `nil`).
#if OS_OBJECT_HAVE_OBJC_SUPPORT
- (instancetype)initWithTarget:(__weak NSObject *)target keyPath:(NSString *)keyPath nilValue:(nullable ValueType)nilValue;
#else
// Swift builds with OS_OBJECT_HAVE_OBJC_SUPPORT=0 for Playgrounds and LLDB :(
- (instancetype)initWithTarget:(NSObject *)target keyPath:(NSString *)keyPath nilValue:(nullable ValueType)nilValue;
#endif

- (instancetype)init __attribute__((unavailable("Use -initWithTarget:keyPath:nilValue: instead")));

@end

/// Methods needed for the convenience macro. Do not call explicitly.
@interface RACKVOChannel (RACChannelTo)

- (RACChannelTerminal *)objectForKeyedSubscript:(NSString *)key;
- (void)setObject:(RACChannelTerminal *)otherTerminal forKeyedSubscript:(NSString *)key;
// Key for the array of RACKVOChannel's additional thread local
// data in the thread dictionary.
static NSString * const RACKVOChannelDataDictionaryKey = @"RACKVOChannelKey";

// Wrapper class for additional thread local data.
@interface RACKVOChannelData : NSObject

// The flag used to ignore updates the channel itself has triggered.
@property (nonatomic, assign) BOOL ignoreNextUpdate;

// A pointer to the owner of the data. Only use this for pointer comparison,
// never as an object reference.
@property (nonatomic, assign) void *owner;

+ (instancetype)dataForChannel:(RACKVOChannel *)channel;

@end

@interface RACKVOChannel ()

// The object whose key path the channel is wrapping.
@property (atomic, weak) NSObject *target;

// The key path the channel is wrapping.
@property (nonatomic, copy, readonly) NSString *keyPath;

// Returns the existing thread local data container or nil if none exists.
@property (nonatomic, strong, readonly) RACKVOChannelData *currentThreadData;

// Creates the thread local data container for the channel.
- (void)createCurrentThreadData;

// Destroy the thread local data container for the channel.
- (void)destroyCurrentThreadData;

@end

@implementation RACKVOChannel

#pragma mark Properties

- (RACKVOChannelData *)currentThreadData {
	NSMutableArray *dataArray = NSThread.currentThread.threadDictionary[RACKVOChannelDataDictionaryKey];

	for (RACKVOChannelData *data in dataArray) {
		if (data.owner == (__bridge void *)self) return data;
	}

	return nil;
}

#pragma mark Lifecycle

- (instancetype)initWithTarget:(__weak NSObject *)target keyPath:(NSString *)keyPath nilValue:(id)nilValue {
	NSCParameterAssert(keyPath.rac_keyPathComponents.count > 0);

	NSObject *strongTarget = target;

	self = [super init];

	_target = target;
	_keyPath = [keyPath copy];

	[self.leadingTerminal setNameWithFormat:@"[-initWithTarget: %@ keyPath: %@ nilValue: %@] -leadingTerminal", target, keyPath, nilValue];
	[self.followingTerminal setNameWithFormat:@"[-initWithTarget: %@ keyPath: %@ nilValue: %@] -followingTerminal", target, keyPath, nilValue];

	if (strongTarget == nil) {
		[self.leadingTerminal sendCompleted];
		return self;
	}

	// Observe the key path on target for changes and forward the changes to the
	// terminal.
	//
	// Intentionally capturing `self` strongly in the blocks below, so the
	// channel object stays alive while observing.
	RACDisposable *observationDisposable = [strongTarget rac_observeKeyPath:keyPath options:NSKeyValueObservingOptionInitial observer:nil block:^(id value, NSDictionary *change, BOOL causedByDealloc, BOOL affectedOnlyLastComponent) {
		// If the change wasn't triggered by deallocation, only affects the last
		// path component, and ignoreNextUpdate is set, then it was triggered by
		// this channel and should not be forwarded.
		if (!causedByDealloc && affectedOnlyLastComponent && self.currentThreadData.ignoreNextUpdate) {
			[self destroyCurrentThreadData];
			return;
		}

		[self.leadingTerminal sendNext:value];
	}];

	NSString *keyPathByDeletingLastKeyPathComponent = keyPath.rac_keyPathByDeletingLastKeyPathComponent;
	NSArray *keyPathComponents = keyPath.rac_keyPathComponents;
	NSUInteger keyPathComponentsCount = keyPathComponents.count;
	NSString *lastKeyPathComponent = keyPathComponents.lastObject;

	// Update the value of the property with the values received.
	[[self.leadingTerminal
		finally:^{
			[observationDisposable dispose];
		}]
		subscribeNext:^(id x) {
			// Check the value of the second to last key path component. Since the
			// channel can only update the value of a property on an object, and not
			// update intermediate objects, it can only update the value of the whole
			// key path if this object is not nil.
			NSObject *object = (keyPathComponentsCount > 1 ? [self.target valueForKeyPath:keyPathByDeletingLastKeyPathComponent] : self.target);
			if (object == nil) return;

			// Set the ignoreNextUpdate flag before setting the value so this channel
			// ignores the value in the subsequent -didChangeValueForKey: callback.
			[self createCurrentThreadData];
			self.currentThreadData.ignoreNextUpdate = YES;

			[object setValue:x ?: nilValue forKey:lastKeyPathComponent];
		} error:^(NSError *error) {
			NSCAssert(NO, @"Received error in %@: %@", self, error);

			// Log the error if we're running with assertions disabled.
			NSLog(@"Received error in %@: %@", self, error);
		}];

	// Capture `self` weakly for the target's deallocation disposable, so we can
	// freely deallocate if we complete before then.
	@weakify(self);

	[strongTarget.rac_deallocDisposable addDisposable:[RACDisposable disposableWithBlock:^{
		@strongify(self);
		[self.leadingTerminal sendCompleted];
		self.target = nil;
	}]];

	return self;
}

- (void)createCurrentThreadData {
	NSMutableArray *dataArray = NSThread.currentThread.threadDictionary[RACKVOChannelDataDictionaryKey];
	if (dataArray == nil) {
		dataArray = [NSMutableArray array];
		NSThread.currentThread.threadDictionary[RACKVOChannelDataDictionaryKey] = dataArray;
		[dataArray addObject:[RACKVOChannelData dataForChannel:self]];
		return;
	}

	for (RACKVOChannelData *data in dataArray) {
		if (data.owner == (__bridge void *)self) return;
	}

	[dataArray addObject:[RACKVOChannelData dataForChannel:self]];
}

- (void)destroyCurrentThreadData {
	NSMutableArray *dataArray = NSThread.currentThread.threadDictionary[RACKVOChannelDataDictionaryKey];
	NSUInteger index = [dataArray indexOfObjectPassingTest:^ BOOL (RACKVOChannelData *data, NSUInteger idx, BOOL *stop) {
		return data.owner == (__bridge void *)self;
	}];

	if (index != NSNotFound) [dataArray removeObjectAtIndex:index];
}

@end

@implementation RACKVOChannel (RACChannelTo)

- (RACChannelTerminal *)objectForKeyedSubscript:(NSString *)key {
	NSCParameterAssert(key != nil);

	RACChannelTerminal *terminal = [self valueForKey:key];
	NSCAssert([terminal isKindOfClass:RACChannelTerminal.class], @"Key \"%@\" does not identify a channel terminal", key);

	return terminal;
}

- (void)setObject:(RACChannelTerminal *)otherTerminal forKeyedSubscript:(NSString *)key {
	NSCParameterAssert(otherTerminal != nil);

	RACChannelTerminal *selfTerminal = [self objectForKeyedSubscript:key];
	[otherTerminal subscribe:selfTerminal];
	[[selfTerminal skip:1] subscribe:otherTerminal];
}

@end

@implementation RACKVOChannelData

+ (instancetype)dataForChannel:(RACKVOChannel *)channel {
	RACKVOChannelData *data = [[self alloc] init];
	data->_owner = (__bridge void *)channel;
	return data;
}

36. RACKVOProxy : NSObject

/// Returns the singleton KVO proxy object.
+ (instancetype)sharedProxy;

/// Registers an observer with the proxy, such that when the proxy receives a
/// KVO change with the given context, it forwards it to the observer.
///
/// observer - True observer of the KVO change. Must not be nil.
/// context  - Arbitrary context object used to differentiate multiple
///            observations of the same keypath. Must be unique, cannot be nil.
- (void)addObserver:(__weak NSObject *)observer forContext:(void *)context;

/// Removes an observer from the proxy. Parameters must match those passed to
/// addObserver:forContext:.
///
/// observer - True observer of the KVO change. Must not be nil.
/// context  - Arbitrary context object used to differentiate multiple
///            observations of the same keypath. Must be unique, cannot be nil.
- (void)removeObserver:(NSObject *)observer forContext:(void *)context;

@interface RACKVOProxy()

@property (strong, nonatomic, readonly) NSMapTable *trampolines;
@property (strong, nonatomic, readonly) dispatch_queue_t queue;

@end

@implementation RACKVOProxy

+ (instancetype)sharedProxy {
	static RACKVOProxy *proxy;
	static dispatch_once_t onceToken;

	dispatch_once(&onceToken, ^{
		proxy = [[self alloc] init];
	});

	return proxy;
}

- (instancetype)init {
	self = [super init];

	_queue = dispatch_queue_create("org.reactivecocoa.ReactiveObjC.RACKVOProxy", DISPATCH_QUEUE_SERIAL);
	_trampolines = [NSMapTable strongToWeakObjectsMapTable];

	return self;
}

- (void)addObserver:(__weak NSObject *)observer forContext:(void *)context {
	NSValue *valueContext = [NSValue valueWithPointer:context];

	dispatch_sync(self.queue, ^{
		[self.trampolines setObject:observer forKey:valueContext];
	});
}

- (void)removeObserver:(NSObject *)observer forContext:(void *)context {
	NSValue *valueContext = [NSValue valueWithPointer:context];

	dispatch_sync(self.queue, ^{
		[self.trampolines removeObjectForKey:valueContext];
	});
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
	NSValue *valueContext = [NSValue valueWithPointer:context];
	__block NSObject *trueObserver;

	dispatch_sync(self.queue, ^{
		trueObserver = [self.trampolines objectForKey:valueContext];
	});

	if (trueObserver != nil) {
		[trueObserver observeValueForKeyPath:keyPath ofObject:object change:change context:context];
	}
}

@end
37. RACKVOTrampoline

// Returns the initialized object.
- (instancetype)initWithTarget:(__weak NSObject *)target observer:(__weak NSObject *)observer keyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options block:(RACKVOBlock)block;

@interface RACKVOTrampoline ()

// The keypath which the trampoline is observing.
@property (nonatomic, readonly, copy) NSString *keyPath;

// These properties should only be manipulated while synchronized on the
// receiver.
@property (nonatomic, readonly, copy) RACKVOBlock block;
@property (nonatomic, readonly, unsafe_unretained) NSObject *unsafeTarget;
@property (nonatomic, readonly, weak) NSObject *weakTarget;
@property (nonatomic, readonly, weak) NSObject *observer;

@end

@implementation RACKVOTrampoline

#pragma mark Lifecycle

- (instancetype)initWithTarget:(__weak NSObject *)target observer:(__weak NSObject *)observer keyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options block:(RACKVOBlock)block {
	NSCParameterAssert(keyPath != nil);
	NSCParameterAssert(block != nil);

	NSObject *strongTarget = target;
	if (strongTarget == nil) return nil;

	self = [super init];

	_keyPath = [keyPath copy];

	_block = [block copy];
	_weakTarget = target;
	_unsafeTarget = strongTarget;
	_observer = observer;

	[RACKVOProxy.sharedProxy addObserver:self forContext:(__bridge void *)self];
	[strongTarget addObserver:RACKVOProxy.sharedProxy forKeyPath:self.keyPath options:options context:(__bridge void *)self];

	[strongTarget.rac_deallocDisposable addDisposable:self];
	[self.observer.rac_deallocDisposable addDisposable:self];

	return self;
}

- (void)dealloc {
	[self dispose];
}

#pragma mark Observation

- (void)dispose {
	NSObject *target;
	NSObject *observer;

	@synchronized (self) {
		_block = nil;

		// The target should still exist at this point, because we still need to
		// tear down its KVO observation. Therefore, we can use the unsafe
		// reference (and need to, because the weak one will have been zeroed by
		// now).
		target = self.unsafeTarget;
		observer = self.observer;

		_unsafeTarget = nil;
		_observer = nil;
	}

	[target.rac_deallocDisposable removeDisposable:self];
	[observer.rac_deallocDisposable removeDisposable:self];

	[target removeObserver:RACKVOProxy.sharedProxy forKeyPath:self.keyPath context:(__bridge void *)self];
	[RACKVOProxy.sharedProxy removeObserver:self forContext:(__bridge void *)self];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
	if (context != (__bridge void *)self) {
		[super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
		return;
	}

	RACKVOBlock block;
	id observer;
	id target;

	@synchronized (self) {
		block = self.block;
		observer = self.observer;
		target = self.weakTarget;
	}

	if (block == nil || target == nil) return;

	block(target, observer, change);
}

@end


38. RACQueueScheduler


#pragma mark Lifecycle

- (instancetype)initWithName:(NSString *)name queue:(dispatch_queue_t)queue {
	NSCParameterAssert(queue != NULL);

	self = [super initWithName:name];

	_queue = queue;
#if !OS_OBJECT_USE_OBJC
	dispatch_retain(_queue);
#endif

	return self;
}

#if !OS_OBJECT_USE_OBJC

- (void)dealloc {
	if (_queue != NULL) {
		dispatch_release(_queue);
		_queue = NULL;
	}
}

#endif

#pragma mark Date Conversions

+ (dispatch_time_t)wallTimeWithDate:(NSDate *)date {
	NSCParameterAssert(date != nil);

	double seconds = 0;
	double frac = modf(date.timeIntervalSince1970, &seconds);

	struct timespec walltime = {
		.tv_sec = (time_t)fmin(fmax(seconds, LONG_MIN), LONG_MAX),
		.tv_nsec = (long)fmin(fmax(frac * NSEC_PER_SEC, LONG_MIN), LONG_MAX)
	};

	return dispatch_walltime(&walltime, 0);
}

#pragma mark RACScheduler

- (RACDisposable *)schedule:(void (^)(void))block {
	NSCParameterAssert(block != NULL);

	RACDisposable *disposable = [[RACDisposable alloc] init];

	dispatch_async(self.queue, ^{
		if (disposable.disposed) return;
		[self performAsCurrentScheduler:block];
	});

	return disposable;
}

- (RACDisposable *)after:(NSDate *)date schedule:(void (^)(void))block {
	NSCParameterAssert(date != nil);
	NSCParameterAssert(block != NULL);

	RACDisposable *disposable = [[RACDisposable alloc] init];

	dispatch_after([self.class wallTimeWithDate:date], self.queue, ^{
		if (disposable.disposed) return;
		[self performAsCurrentScheduler:block];
	});

	return disposable;
}

- (RACDisposable *)after:(NSDate *)date repeatingEvery:(NSTimeInterval)interval withLeeway:(NSTimeInterval)leeway schedule:(void (^)(void))block {
	NSCParameterAssert(date != nil);
	NSCParameterAssert(interval > 0.0 && interval < INT64_MAX / NSEC_PER_SEC);
	NSCParameterAssert(leeway >= 0.0 && leeway < INT64_MAX / NSEC_PER_SEC);
	NSCParameterAssert(block != NULL);

	uint64_t intervalInNanoSecs = (uint64_t)(interval * NSEC_PER_SEC);
	uint64_t leewayInNanoSecs = (uint64_t)(leeway * NSEC_PER_SEC);

	dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, self.queue);
	dispatch_source_set_timer(timer, [self.class wallTimeWithDate:date], intervalInNanoSecs, leewayInNanoSecs);
	dispatch_source_set_event_handler(timer, block);
	dispatch_resume(timer);

	return [RACDisposable disposableWithBlock:^{
		dispatch_source_cancel(timer);
	}];
}

39. RACReplaySubject<ValueType> : RACSubject<ValueType>

/// Creates a new replay subject with the given capacity. A capacity of
/// RACReplaySubjectUnlimitedCapacity means values are never trimmed.
+ (instancetype)replaySubjectWithCapacity:(NSUInteger)capacity;


const NSUInteger RACReplaySubjectUnlimitedCapacity = NSUIntegerMax;

@interface RACReplaySubject ()

@property (nonatomic, assign, readonly) NSUInteger capacity;

// These properties should only be modified while synchronized on self.
@property (nonatomic, strong, readonly) NSMutableArray *valuesReceived;
@property (nonatomic, assign) BOOL hasCompleted;
@property (nonatomic, assign) BOOL hasError;
@property (nonatomic, strong) NSError *error;

@end


@implementation RACReplaySubject

#pragma mark Lifecycle

+ (instancetype)replaySubjectWithCapacity:(NSUInteger)capacity {
	return [(RACReplaySubject *)[self alloc] initWithCapacity:capacity];
}

- (instancetype)init {
	return [self initWithCapacity:RACReplaySubjectUnlimitedCapacity];
}

- (instancetype)initWithCapacity:(NSUInteger)capacity {
	self = [super init];
	
	_capacity = capacity;
	_valuesReceived = (capacity == RACReplaySubjectUnlimitedCapacity ? [NSMutableArray array] : [NSMutableArray arrayWithCapacity:capacity]);
	
	return self;
}

#pragma mark RACSignal

- (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {
	RACCompoundDisposable *compoundDisposable = [RACCompoundDisposable compoundDisposable];

	RACDisposable *schedulingDisposable = [RACScheduler.subscriptionScheduler schedule:^{
		@synchronized (self) {
			for (id value in self.valuesReceived) {
				if (compoundDisposable.disposed) return;

				[subscriber sendNext:(value == RACTupleNil.tupleNil ? nil : value)];
			}

			if (compoundDisposable.disposed) return;

			if (self.hasCompleted) {
				[subscriber sendCompleted];
			} else if (self.hasError) {
				[subscriber sendError:self.error];
			} else {
				RACDisposable *subscriptionDisposable = [super subscribe:subscriber];
				[compoundDisposable addDisposable:subscriptionDisposable];
			}
		}
	}];

	[compoundDisposable addDisposable:schedulingDisposable];

	return compoundDisposable;
}

#pragma mark RACSubscriber

- (void)sendNext:(id)value {
	@synchronized (self) {
		[self.valuesReceived addObject:value ?: RACTupleNil.tupleNil];
		[super sendNext:value];
		
		if (self.capacity != RACReplaySubjectUnlimitedCapacity && self.valuesReceived.count > self.capacity) {
			[self.valuesReceived removeObjectsInRange:NSMakeRange(0, self.valuesReceived.count - self.capacity)];
		}
	}
}

- (void)sendCompleted {
	@synchronized (self) {
		self.hasCompleted = YES;
		[super sendCompleted];
	}
}

- (void)sendError:(NSError *)e {
	@synchronized (self) {
		self.hasError = YES;
		self.error = e;
		[super sendError:e];
	}
}

40. RACSignalSequence : RACSequence

// Returns a sequence for enumerating over the given signal.
+ (RACSequence *)sequenceWithSignal:(RACSignal *)signal;


@interface RACSignalSequence ()

// Replays the signal given on initialization.
@property (nonatomic, strong, readonly) RACReplaySubject *subject;

@end

@implementation RACSignalSequence

#pragma mark Lifecycle

+ (RACSequence *)sequenceWithSignal:(RACSignal *)signal {
	RACSignalSequence *seq = [[self alloc] init];

	RACReplaySubject *subject = [RACReplaySubject subject];
	[signal subscribeNext:^(id value) {
		[subject sendNext:value];
	} error:^(NSError *error) {
		[subject sendError:error];
	} completed:^{
		[subject sendCompleted];
	}];

	seq->_subject = subject;
	return seq;
}

#pragma mark RACSequence

- (id)head {
	id value = [self.subject firstOrDefault:self];

	if (value == self) {
		return nil;
	} else {
		return value ?: NSNull.null;
	}
}

- (RACSequence *)tail {
	RACSequence *sequence = [self.class sequenceWithSignal:[self.subject skip:1]];
	sequence.name = self.name;
	return sequence;
}

- (NSArray *)array {
	return self.subject.toArray;
}

#pragma mark NSObject

- (NSString *)description {
	// Synchronously accumulate the values that have been sent so far.
	NSMutableArray *values = [NSMutableArray array];
	RACDisposable *disposable = [self.subject subscribeNext:^(id value) {
		@synchronized (values) {
			[values addObject:value ?: NSNull.null];
		}
	}];

	[disposable dispose];

	return [NSString stringWithFormat:@"<%@: %p>{ name = %@, values = %@ … }", self.class, self, self.name, values];
}


41. RACTestScheduler : RACScheduler

/// Initializes a new test scheduler.
- (instancetype)init;

/// Executes the next scheduled block, if any.
///
/// This method will block until the scheduled action has completed.
- (void)step;

/// Executes up to the next `ticks` scheduled blocks.
///
/// This method will block until the scheduled actions have completed.
///
/// ticks - The number of scheduled blocks to execute. If there aren't this many
///         blocks enqueued, all scheduled blocks are executed.
- (void)step:(NSUInteger)ticks;

/// Executes all of the scheduled blocks on the receiver.
///
/// This method will block until the scheduled actions have completed.
- (void)stepAll;

@interface RACTestSchedulerAction : NSObject

// The date at which the action should be executed.
//
// This absolute time will not actually be honored. This date is only used for
// comparison, to determine which block should be run _next_.
@property (nonatomic, copy, readonly) NSDate *date;

// The scheduled block.
@property (nonatomic, copy, readonly) void (^block)(void);

// A disposable for this action.
//
// When disposed, the action should not start executing if it hasn't already.
@property (nonatomic, strong, readonly) RACDisposable *disposable;

// Initializes a new scheduler action.
- (instancetype)initWithDate:(NSDate *)date block:(void (^)(void))block;

@end

static CFComparisonResult RACCompareScheduledActions(const void *ptr1, const void *ptr2, void *info) {
	RACTestSchedulerAction *action1 = (__bridge id)ptr1;
	RACTestSchedulerAction *action2 = (__bridge id)ptr2;
	return CFDateCompare((__bridge CFDateRef)action1.date, (__bridge CFDateRef)action2.date, NULL);
}

static const void *RACRetainScheduledAction(CFAllocatorRef allocator, const void *ptr) {
	return CFRetain(ptr);
}

static void RACReleaseScheduledAction(CFAllocatorRef allocator, const void *ptr) {
	CFRelease(ptr);
}

@interface RACTestScheduler ()

// All of the RACTestSchedulerActions that have been enqueued and not yet
// executed.
//
// The minimum value in the heap represents the action to execute next.
//
// This property should only be used while synchronized on self.
@property (nonatomic, assign, readonly) CFBinaryHeapRef scheduledActions;

// The number of blocks that have been directly enqueued with -schedule: so
// far.
//
// This is used to ensure unique dates when two blocks are enqueued
// simultaneously.
//
// This property should only be used while synchronized on self.
@property (nonatomic, assign) NSUInteger numberOfDirectlyScheduledBlocks;

@end

@implementation RACTestScheduler

#pragma mark Lifecycle

- (instancetype)init {
	self = [super initWithName:@"org.reactivecocoa.ReactiveObjC.RACTestScheduler"];

	CFBinaryHeapCallBacks callbacks = (CFBinaryHeapCallBacks){
		.version = 0,
		.retain = &RACRetainScheduledAction,
		.release = &RACReleaseScheduledAction,
		.copyDescription = &CFCopyDescription,
		.compare = &RACCompareScheduledActions
	};

	_scheduledActions = CFBinaryHeapCreate(NULL, 0, &callbacks, NULL);
	return self;
}

- (void)dealloc {
	[self stepAll];

	if (_scheduledActions != NULL) {
		CFBridgingRelease(_scheduledActions);
		_scheduledActions = NULL;
	}
}

#pragma mark Execution

- (void)step {
	[self step:1];
}

- (void)step:(NSUInteger)ticks {
	@synchronized (self) {
		for (NSUInteger i = 0; i < ticks; i++) {
			const void *actionPtr = NULL;
			if (!CFBinaryHeapGetMinimumIfPresent(self.scheduledActions, &actionPtr)) break;

			RACTestSchedulerAction *action = (__bridge id)actionPtr;
			CFBinaryHeapRemoveMinimumValue(self.scheduledActions);

			if (action.disposable.disposed) continue;

			RACScheduler *previousScheduler = RACScheduler.currentScheduler;
			NSThread.currentThread.threadDictionary[RACSchedulerCurrentSchedulerKey] = self;

			action.block();

			if (previousScheduler != nil) {
				NSThread.currentThread.threadDictionary[RACSchedulerCurrentSchedulerKey] = previousScheduler;
			} else {
				[NSThread.currentThread.threadDictionary removeObjectForKey:RACSchedulerCurrentSchedulerKey];
			}
		}
	}
}

- (void)stepAll {
	[self step:NSUIntegerMax];
}

#pragma mark RACScheduler

- (RACDisposable *)schedule:(void (^)(void))block {
	NSCParameterAssert(block != nil);

	@synchronized (self) {
		NSDate *uniqueDate = [NSDate dateWithTimeIntervalSinceReferenceDate:self.numberOfDirectlyScheduledBlocks];
		self.numberOfDirectlyScheduledBlocks++;

		RACTestSchedulerAction *action = [[RACTestSchedulerAction alloc] initWithDate:uniqueDate block:block];
		CFBinaryHeapAddValue(self.scheduledActions, (__bridge void *)action);

		return action.disposable;
	}
}

- (RACDisposable *)after:(NSDate *)date schedule:(void (^)(void))block {
	NSCParameterAssert(date != nil);
	NSCParameterAssert(block != nil);

	@synchronized (self) {
		RACTestSchedulerAction *action = [[RACTestSchedulerAction alloc] initWithDate:date block:block];
		CFBinaryHeapAddValue(self.scheduledActions, (__bridge void *)action);

		return action.disposable;
	}
}

- (RACDisposable *)after:(NSDate *)date repeatingEvery:(NSTimeInterval)interval withLeeway:(NSTimeInterval)leeway schedule:(void (^)(void))block {
	NSCParameterAssert(date != nil);
	NSCParameterAssert(block != nil);
	NSCParameterAssert(interval >= 0);
	NSCParameterAssert(leeway >= 0);

	RACCompoundDisposable *compoundDisposable = [RACCompoundDisposable compoundDisposable];

	@weakify(self);
	@synchronized (self) {
		__block RACDisposable *thisDisposable = nil;

		void (^reschedulingBlock)(void) = ^{
			@strongify(self);

			[compoundDisposable removeDisposable:thisDisposable];

			// Schedule the next interval.
			RACDisposable *schedulingDisposable = [self after:[date dateByAddingTimeInterval:interval] repeatingEvery:interval withLeeway:leeway schedule:block];
			[compoundDisposable addDisposable:schedulingDisposable];

			block();
		};

		RACTestSchedulerAction *action = [[RACTestSchedulerAction alloc] initWithDate:date block:reschedulingBlock];
		CFBinaryHeapAddValue(self.scheduledActions, (__bridge void *)action);

		thisDisposable = action.disposable;
		[compoundDisposable addDisposable:thisDisposable];
	}

	return compoundDisposable;
}

@end

@implementation RACTestSchedulerAction

#pragma mark Lifecycle

- (instancetype)initWithDate:(NSDate *)date block:(void (^)(void))block {
	NSCParameterAssert(date != nil);
	NSCParameterAssert(block != nil);

	self = [super init];

	_date = [date copy];
	_block = [block copy];
	_disposable = [[RACDisposable alloc] init];

	return self;
}

#pragma mark NSObject

- (NSString *)description {
	return [NSString stringWithFormat:@"<%@: %p>{ date: %@ }", self.class, self, self.date];
}

@end


42. RACValueTransformer : NSValueTransformer

+ (instancetype)transformerWithBlock:(id (^)(id value))block;
@interface RACValueTransformer ()
@property (nonatomic, copy) id (^transformBlock)(id value);
@end


@implementation RACValueTransformer


#pragma mark NSValueTransformer

+ (BOOL)allowsReverseTransformation {
	return NO;
}

- (id)transformedValue:(id)value {
    return self.transformBlock(value);
}


#pragma mark API

@synthesize transformBlock;

+ (instancetype)transformerWithBlock:(id (^)(id value))block {
	NSCParameterAssert(block != NULL);
	
	RACValueTransformer *transformer = [[self alloc] init];
	transformer.transformBlock = block;
	return transformer;
}

@end
43.

44.

45.

46.

47.

48.

49.

50.


## Protocol
1. RACSubscriber

    - (void)sendNext:(nullable id)value;

    - (void)sendError:(nullable NSError *)error;

    - (void)sendCompleted;

    - (void)didSubscribeWithDisposable:(RACCompoundDisposable *)disposable;

## Category 

1. NSArray + RACSequenceAddtions
    // return [RACArraySequence sequenceWithArray:self offset:0];
    @property (nonatomic, copy, readonly) RACSequence<ObjectType> *rac_sequence;

2. NSControl (RACCommandSupport)
    // return objc_getAssociatedObject(self, NSControlRACCommandKey);
    @property (nonatomic, strong, nullable) RACCommand<__kindof NSControl *, id> *rac_command;
    // .m
    /*
    setAssocation command 然后 NSControlEnabledDisposableKey dispose  NSControlEnabledDisposableKey nil
    如果command为nil self.enable = yes rac_hijackActionAndTargetIfNeeded 
    */  
    - (void)setRac_command:(RACCommand *)command { }

    - (void)rac_hijackActionAndTargetIfNeeded { }

    - (void)rac_commandPerformAction:(id)sender {  }

3. @interface NSControl (RACTextSignalSupport)

    - (RACSignal<NSString *> *)rac_textSignal;

4. NSData (RACSupport)

     //
    + (RACSignal<NSData *> *)rac_readContentsOfURL:(nullable NSURL *)URL options:(NSDataReadingOptions)options scheduler:(RACScheduler *)scheduler; 

5. NSDictionary<__covariant KeyType, __covariant ObjectType> (RACSequenceAdditions)
 
    @property (nonatomic, copy, readonly) RACSequence<RACTwoTuple<KeyType, ObjectType> *> *rac_sequence;
    
    @property (nonatomic, copy, readonly) RACSequence<KeyType> *rac_keySequence;
    
    @property (nonatomic, copy, readonly) RACSequence<ObjectType> *rac_valueSequence;

6. NSEnumerator<ObjectType> (RACSequenceAdditions)

    @property (nonatomic, copy, readonly) RACSequence<ObjectType> *rac_sequence;

7. NSFileHandle (RACSupport)

    // Read any available data in the background and send it. Completes when data
    // length is <= 0.
    - (RACSignal<NSData *> *)rac_readInBackground;


8. NSIndexSet (RACSequenceAdditions)

/// Creates and returns a sequence of indexes (as `NSNumber`s) corresponding to
/// the receiver.
///
/// Mutating the receiver will not affect the sequence after it's been created.
@property (nonatomic, copy, readonly) RACSequence<NSNumber *> *rac_sequence;


9. NSInvocation (RACTypeParsing)

// Sets the argument for the invocation at the given index by unboxing the given
// object based on the type signature of the argument.
//
// This does not support C arrays or unions.
//
// Note that calling this on a char * or const char * argument can cause all
// arguments to be retained.
//
// object - The object to unbox and set as the argument.
// index  - The index of the argument to set.
- (void)rac_setArgument:(id)object atIndex:(NSUInteger)index;

// Gets the argument for the invocation at the given index based on the
// invocation's method signature. The value is then wrapped in the appropriate
// object type.
//
// This does not support C arrays or unions.
//
// index  - The index of the argument to get.
//
// Returns the argument of the invocation, wrapped in an object.
- (id)rac_argumentAtIndex:(NSUInteger)index;

// Arguments tuple for the invocation.
//
// The arguments tuple excludes implicit variables `self` and `_cmd`.
//
// See -rac_argumentAtIndex: and -rac_setArgumentAtIndex: for further
// description of the underlying behavior.
@property (nonatomic, copy) RACTuple *rac_argumentsTuple;

// Gets the return value from the invocation based on the invocation's method
// signature. The value is then wrapped in the appropriate object type.
//
// This does not support C arrays or unions.
//
// Returns the return value of the invocation, wrapped in an object. Voids are
// returned as `RACUnit.defaultUnit`.
- (id)rac_returnValue;

10. NSNotificationCenter (RACSupport)

// Sends the NSNotification every time the notification is posted.
- (RACSignal<NSNotification *> *)rac_addObserverForName:(nullable NSString *)notificationName object:(nullable id)object;


11. NSObject (RACAppKitBindings)

/// Invokes -rac_channelToBinding:options: without any options.
- (RACChannelTerminal *)rac_channelToBinding:(NSString *)binding;

/// Applies a Cocoa binding to the receiver, then exposes a RACChannel-based
/// interface for manipulating it.
///
/// Creating two of the same bindings on the same object will result in undefined
/// behavior.
///
/// binding - The name of the binding. This must not be nil.
/// options - Any options to pass to Cocoa Bindings. This may be nil.
///
/// Returns a RACChannelTerminal which will send future values from the receiver,
/// and update the receiver when values are sent to the terminal.
- (RACChannelTerminal *)rac_channelToBinding:(NSString *)binding options:(nullable NSDictionary *)options;


12. NSObject (RACDeallocating)

/// The compound disposable which will be disposed of when the receiver is
/// deallocated.
@property (atomic, readonly, strong) RACCompoundDisposable *rac_deallocDisposable;

/// Returns a signal that will complete immediately before the receiver is fully
/// deallocated. If already deallocated when the signal is subscribed to,
/// a `completed` event will be sent immediately.
- (RACSignal *)rac_willDeallocSignal;

13. NSObject + RACDescription

    NSString *RACDescription(id object);

14. NSObject (RACKVOWrapper)

// Adds the given block as the callbacks for when the key path changes.
//
// Unlike direct KVO observation, this handles deallocation of `weak` properties
// by generating an appropriate notification. This will only occur if there is
// an `@property` declaration visible in the observed class, with the `weak`
// memory management attribute.
//
// The observation does not need to be explicitly removed. It will be removed
// when the observer or the receiver deallocate.
//
// keyPath  - The key path to observe. Must not be nil.
// options  - The KVO observation options.
// observer - The object that requested the observation. May be nil.
// block    - The block called when the value at the key path changes. It is
//            passed the current value of the key path and the extended KVO
//            change dictionary including RAC-specific keys and values. Must not
//            be nil.
//
// Returns a disposable that can be used to stop the observation.
- (RACDisposable *)rac_observeKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options observer:(__weak NSObject *)observer block:(void (^)(id value, NSDictionary *change, BOOL causedByDealloc, BOOL affectedOnlyLastComponent))block;

@end

typedef void (^RACKVOBlock)(id target, id observer, NSDictionary *change);

@interface NSObject (RACUnavailableKVOWrapper)

- (RACKVOTrampoline *)rac_addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options block:(RACKVOBlock)block __attribute((unavailable("Use rac_observeKeyPath:options:observer:block: instead.")));


15. NSObject (RACLifting)

/// Lifts the selector on the receiver into the reactive world. The selector will
/// be invoked whenever any signal argument sends a value, but only after each
/// signal has sent an initial value.
///
/// It will replay the most recently sent value to new subscribers.
///
/// This does not support C arrays or unions.
///
/// selector    - The selector on self to invoke.
/// firstSignal - The signal corresponding to the first method argument. This
///               must not be nil.
/// ...         - A list of RACSignals corresponding to the remaining arguments.
///               There must be a non-nil signal for each method argument.
///
/// Examples
///
///   [button rac_liftSelector:@selector(setTitleColor:forState:) withSignals:textColorSignal, [RACSignal return:@(UIControlStateNormal)], nil];
///
/// Returns a signal which sends the return value from each invocation of the
/// selector. If the selector returns void, it instead sends RACUnit.defaultUnit.
/// It completes only after all the signal arguments complete.
- (RACSignal *)rac_liftSelector:(SEL)selector withSignals:(RACSignal *)firstSignal, ... NS_REQUIRES_NIL_TERMINATION;

/// Like -rac_liftSelector:withSignals:, but accepts an array instead of
/// a variadic list of arguments.
- (RACSignal *)rac_liftSelector:(SEL)selector withSignalsFromArray:(NSArray<RACSignal *> *)signals;

/// Like -rac_liftSelector:withSignals:, but accepts a signal sending tuples of
/// arguments instead of a variadic list of arguments.
- (RACSignal *)rac_liftSelector:(SEL)selector withSignalOfArguments:(RACSignal<RACTuple *> *)arguments;


16. #define _RACObserve(TARGET, KEYPATH) \
({ \
	__weak id target_ = (TARGET); \
	[target_ rac_valuesForKeyPath:@keypath(TARGET, KEYPATH) observer:self]; \
})

#if __clang__ && (__clang_major__ >= 8)
#define RACObserve(TARGET, KEYPATH) _RACObserve(TARGET, KEYPATH)
#else
#define RACObserve(TARGET, KEYPATH) \
({ \
	_Pragma("clang diagnostic push") \
	_Pragma("clang diagnostic ignored \"-Wreceiver-is-weak\"") \
	_RACObserve(TARGET, KEYPATH) \
	_Pragma("clang diagnostic pop") \
})
#endif

@class RACDisposable;
@class RACTwoTuple<__covariant First, __covariant Second>;
@class RACSignal<__covariant ValueType>;

NS_ASSUME_NONNULL_BEGIN

@interface NSObject (RACPropertySubscribing)

/// Creates a signal to observe the value at the given key path.
///
/// The initial value is sent on subscription, the subsequent values are sent
/// from whichever thread the change occured on, even if it doesn't have a valid
/// scheduler.
///
/// Returns a signal that immediately sends the receiver's current value at the
/// given keypath, then any changes thereafter.
#if OS_OBJECT_HAVE_OBJC_SUPPORT
- (RACSignal *)rac_valuesForKeyPath:(NSString *)keyPath observer:(__weak NSObject *)observer;
#else
// Swift builds with OS_OBJECT_HAVE_OBJC_SUPPORT=0 for Playgrounds and LLDB :(
- (RACSignal *)rac_valuesForKeyPath:(NSString *)keyPath observer:(NSObject *)observer;
#endif

/// Creates a signal to observe the changes of the given key path.
///
/// The initial value is sent on subscription if `NSKeyValueObservingOptionInitial` is set.
/// The subsequent values are sent from whichever thread the change occured on,
/// even if it doesn't have a valid scheduler.
///
/// Returns a signal that sends tuples containing the current value at the key
/// path and the change dictionary for each KVO callback.
#if OS_OBJECT_HAVE_OBJC_SUPPORT
- (RACSignal<RACTwoTuple<id, NSDictionary *> *> *)rac_valuesAndChangesForKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options observer:(__weak NSObject *)observer;
#else
- (RACSignal<RACTwoTuple<id, NSDictionary *> *> *)rac_valuesAndChangesForKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options observer:(NSObject *)observer;
#endif

@end

NS_ASSUME_NONNULL_END

#define RACAble(...) \
	metamacro_if_eq(1, metamacro_argcount(__VA_ARGS__)) \
		(_RACAbleObject(self, __VA_ARGS__)) \
		(_RACAbleObject(__VA_ARGS__))

#define _RACAbleObject(object, property) [object rac_signalForKeyPath:@keypath(object, property) observer:self]

#define RACAbleWithStart(...) \
	metamacro_if_eq(1, metamacro_argcount(__VA_ARGS__)) \
		(_RACAbleWithStartObject(self, __VA_ARGS__)) \
		(_RACAbleWithStartObject(__VA_ARGS__))

#define _RACAbleWithStartObject(object, property) [object rac_signalWithStartingValueForKeyPath:@keypath(object, property) observer:self]


17. 
/// The domain for any errors originating from -rac_signalForSelector:.
extern NSErrorDomain const RACSelectorSignalErrorDomain;

typedef NS_ERROR_ENUM(RACSelectorSignalErrorDomain, RACSelectorSignalError) {
	/// -rac_signalForSelector: was going to add a new method implementation for
	/// `selector`, but another thread added an implementation before it was able to.
	///
	/// This will _not_ occur for cases where a method implementation exists before
	/// -rac_signalForSelector: is invoked.
	RACSelectorSignalErrorMethodSwizzlingRace = 1,
};

@interface NSObject (RACSelectorSignal)

/// Creates a signal associated with the receiver, which will send a tuple of the
/// method's arguments each time the given selector is invoked.
///
/// If the selector is already implemented on the receiver, the existing
/// implementation will be invoked _before_ the signal fires.
///
/// If the selector is not yet implemented on the receiver, the injected
/// implementation will have a `void` return type and accept only object
/// arguments. Invoking the added implementation with non-object values, or
/// expecting a return value, will result in undefined behavior.
///
/// This is useful for changing an event or delegate callback into a signal. For
/// example, on an NSView:
///
///     [[view rac_signalForSelector:@selector(mouseDown:)] subscribeNext:^(RACTuple *args) {
///         NSEvent *event = args.first;
///         NSLog(@"mouse button pressed: %@", event);
///     }];
///
/// selector - The selector for whose invocations are to be observed. If it
///            doesn't exist, it will be implemented to accept object arguments
///            and return void. This cannot have C arrays or unions as arguments
///            or C arrays, unions, structs, complex or vector types as return
///            type.
///
/// Returns a signal which will send a tuple of arguments upon each invocation of
/// the selector, then completes when the receiver is deallocated. `next` events
/// will be sent synchronously from the thread that invoked the method. If
/// a runtime call fails, the signal will send an error in the
/// RACSelectorSignalErrorDomain.
- (RACSignal<RACTuple *> *)rac_signalForSelector:(SEL)selector;

/// Behaves like -rac_signalForSelector:, but if the selector is not yet
/// implemented on the receiver, its method signature is looked up within
/// `protocol`, and may accept non-object arguments.
///
/// If the selector is not yet implemented and has a return value, the injected
/// method will return all zero bits (equal to `nil`, `NULL`, 0, 0.0f, etc.).
///
/// selector - The selector for whose invocations are to be observed. If it
///            doesn't exist, it will be implemented using information from
///            `protocol`, and may accept non-object arguments and return
///            a value. This cannot have C arrays or unions as arguments or
///            return type.
/// protocol - The protocol in which `selector` is declared. This will be used
///            for type information if the selector is not already implemented on
///            the receiver. This must not be `NULL`, and `selector` must exist
///            in this protocol.
///
/// Returns a signal which will send a tuple of arguments on each invocation of
/// the selector, or an error in RACSelectorSignalErrorDomain if a runtime
/// call fails.
- (RACSignal<RACTuple *> *)rac_signalForSelector:(SEL)selector fromProtocol:(Protocol *)protocol;


18. NSOrderedSet<__covariant ObjectType> (RACSequenceAdditions)

/// Creates and returns a sequence corresponding to the receiver.
///
/// Mutating the receiver will not affect the sequence after it's been created.
@property (nonatomic, copy, readonly) RACSequence<ObjectType> *rac_sequence;

19. NSSet<__covariant ObjectType> (RACSequenceAdditions)

/// Creates and returns a sequence corresponding to the receiver.
///
/// Mutating the receiver will not affect the sequence after it's been created.
@property (nonatomic, copy, readonly) RACSequence<ObjectType> *rac_sequence;

20. NSString (RACKeyPathUtilities)

// Returns an array of the components of the receiver.
//
// Calling this method on a string that isn't a key path is considered undefined
// behavior.
- (NSArray *)rac_keyPathComponents;

// Returns a key path with all the components of the receiver except for the
// last one.
//
// Calling this method on a string that isn't a key path is considered undefined
// behavior.
- (NSString *)rac_keyPathByDeletingLastKeyPathComponent;

// Returns a key path with all the components of the receiver expect for the
// first one.
//
// Calling this method on a string that isn't a key path is considered undefined
// behavior.
- (NSString *)rac_keyPathByDeletingFirstKeyPathComponent;


21. NSString (RACSequenceAdditions)

/// Creates and returns a sequence containing strings corresponding to each
/// composed character sequence in the receiver.
///
/// Mutating the receiver will not affect the sequence after it's been created.
@property (nonatomic, copy, readonly) RACSequence<NSString *> *rac_sequence;

22. NSString (RACSupport)

// Reads in the contents of the file using +[NSString stringWithContentsOfURL:usedEncoding:error:].
// Note that encoding won't be valid until the signal completes successfully.
//
// scheduler - cannot be nil.
+ (RACSignal<NSString *> *)rac_readContentsOfURL:(NSURL *)URL usedEncoding:(NSStringEncoding *)encoding scheduler:(RACScheduler *)scheduler;


23. NSText (RACSignalSupport)

/// Returns a signal which sends the current `string` of the receiver, then the
/// new value any time it changes.
- (RACSignal<NSString *> *)rac_textSignal;

24. NSURLConnection (RACSupport)

/// Lazily loads data for the given request in the background.
///
/// request - The URL request to load. This must not be nil.
///
/// Returns a signal which will begin loading the request upon each subscription,
/// then send a tuple of the received response and downloaded data, and complete
/// on a background thread. If any errors occur, the returned signal will error
/// out.
+ (RACSignal<RACTwoTuple<NSURLResponse *, NSData *> *> *)rac_sendAsynchronousRequest:(NSURLRequest *)request;

25. NSUserDefaults (RACSupport)

/// Creates and returns a terminal for binding the user defaults key.
///
/// **Note:** The value in the user defaults is *asynchronously* updated with
/// values sent to the channel.
///
/// key - The user defaults key to create the channel terminal for.
///
/// Returns a channel terminal that sends the value of the user defaults key
/// upon subscription, sends an updated value whenever the default changes, and
/// updates the default asynchronously with values it receives.
- (RACChannelTerminal *)rac_channelTerminalForKey:(NSString *)key;

26.

27.

28.

29.

30.