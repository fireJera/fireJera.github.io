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

    /// Creates and returns a new compound disposable.
    + (instancetype)compoundDisposable;

    /// Creates and returns a new compound disposable containing the given
    /// disposables.
    + (instancetype)compoundDisposableWithDisposables:(nullable NSArray *)disposables;

    /// Adds the given disposable. If the receiving disposable has already been
    /// disposed of, the given disposable is disposed immediately.
    ///
    /// This method is thread-safe.
    ///
    /// disposable - The disposable to add. This may be nil, in which case nothing
    ///              happens.
    - (void)addDisposable:(nullable RACDisposable *)disposable;

    /// Removes the specified disposable from the compound disposable (regardless of
    /// its disposed status), or does nothing if it's not in the compound disposable.
    ///
    /// This is mainly useful for limiting the memory usage of the compound
    /// disposable for long-running operations.
    ///
    /// This method is thread-safe.
    ///
    /// disposable - The disposable to remove. This argument may be nil (to make the
    ///              use of weak references easier).
    - (void)removeDisposable:(nullable RACDisposable *)disposable;


    .m

    #define RACCompoundDisposableInlineCount 2

    // 创建一个static CFMutableArray
    static CFMutableArrayRef RACCreateDisposablesArray(void) {
        // Compare values using only pointer equality.
        CFArrayCallBacks callbacks = kCFTypeArrayCallBacks;
        callbacks.equal = NULL;

        return CFArrayCreateMutable(NULL, 0, &callbacks);
    }
    // 实例变量
    {
        // Used for synchronization.
        pthread_mutex_t _mutex;

        #if RACCompoundDisposableInlineCount

        RACDisposable *_inlineDisposables[RACCompoundDisposableInlineCount];
        #endif

        CFMutableArrayRef _disposables;

        BOOL _disposed;
    }


    - (BOOL)isDisposed {}

    #pragma mark Lifecycle

    + (instancetype)compoundDisposable {}

    + (instancetype)compoundDisposableWithDisposables:(NSArray *)disposables {return [[self alloc] initWithDisposables:disposables];}
    
    // 初始化_mutex
    - (instancetype)init {
        const int result __attribute__((unused)) = pthread_mutex_init(&_mutex, NULL);
        NSCAssert(0 == result, @"Failed to initialize mutex with error %d.", result);
    }

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


## Protocol
1. RACSubscriber

    - (void)sendNext:(nullable id)value;

    - (void)sendError:(nullable NSError *)error;

    - (void)sendCompleted;

    - (void)didSubscribeWithDisposable:(RACCompoundDisposable *)disposable;