---
title: swift or OC
descrition: Even if we alerady have Objective-C to develop ios App, why Apple still publish swift.What's difference.
header: swift or OC
---

All glory comes from daring to begin.

swift's charm:
simple code result in read easily,maintaion easily.
safer(type safe)
ARC(memory management)
less name collisions(implici namespace)
playground(see result in time)

There is most significant point: perfomance


在CPU负荷较大的Mandelbrot的测试中，Swift取得了与C++相近的成绩。
在GEMM测试中（侧重于大数据在有限内存中顺序读取操作），Swift与C++差距变大了。
在FFT测试中（大数组随机读取），C++取得的成绩是Swift的近10倍。
因此可以看出，从运行效率上看，Swift不能完全胜任所有的场景。

综上所述：Swift在代码效率的3各方面，虽然有一定优势，但是还不能由此得出“我们应该转向Swift”的结论。

What can we conclude from these results? The Mandebrot results indicate Swift's strong potential for compute-intensive code while the GEMM and FFT results show the care that must be exercised. GEMM suggests that the Swift compiler cannot vectorize code that the C++ compiler can vectorize, leaving some easy performance gains behind. FFT suggests that developers should reduce calls to instance methods, or should favor an iterative approach over a recursive approach.

Swift is still a young language with a new compiler so we can expect significant improvements to both the compiler and the optimizer in the future. If you're considering writing performance-critical code in Swift today it's certainly worth writing the code in Swift before dropping down to C++. It might just turn out to be fast enough.

Here are the results of our comparison.
Pros:
Swift has a lot of cool things, such as safe memory management, strong typing, generics and optionals, simple but strict inheritance rules.
Swift is cleaner and more readable than Objective-C. There are modules that eliminate class prefixes. It also has half as many files in a project, and understandable closure syntax.
Swift allows to create flexible and lightweight classes which contain exactly what you want (no root class), i.e. if you want to print description, just implement the protocol Printable and if you want to compare -  implement Comparable.
Swift isn’t that fast, but isn’t slower than Objective-C either.
Cons:
Compiler provides misleading and confusing errors (while typecasting and implementing generics, for example).
Poor third party IDE support – AppCode still doesn’t have code generating and on-the-fly analysis, not mentioning simple code completion.
Native IDE - XCode has a lot of bugs. For example, the latest released XCode 6.3 version doesn’t recognize Swift’s unit tests, so UI test buttons don’t come up. What’s even more disappointing is that this function worked in Beta 4.


First, (IMO) comparing with Python is nearly meaningless. Only comparison with Objective-C is meaningful.

How can a new programming language be so much faster?
Objective-C is a slow language. (Only C part is fast, but that's because it's C) It has never been extremely fast. It was just fast enough for their (Apple's) purpose, and faster then their older versions. And it was slow because...

Do the Objective-C results from a bad compiler or is there something less efficient in Objective-C than Swift?
Objective-C guaranteed every method to be dynamically dispatched. No static dispatch at all. That made it impossible to optimize an Objective-C program further. Well, maybe JIT technology can be some help, but AFAIK, Apple really hate unpredictable performance characteristics and object lifetime. I don't think they had adopted any JIT stuff. Swift doesn't have such dynamic dispatch guarantee unless you put some special attribute for Objective-C compatibility.

How would you explain a 40% performance increase? I understand that garbage collection/automated reference control might produce some additional overhead, but this much?
GC or RC doesn't matter here. Swift is also employing RC primarily. No GC is there, and also will not unless there's some huge architectural leap on GC technology. (IMO, it's forever) I believe Swift has a lot more room for static optimization. Especially low level encryption algorithms, as they usually rely on huge numeric calculations, and this is a huge win for statically dispatch languages.

Actually I was surprised because 40% seems too small. I expected far more. Anyway, this is the initial release, and I think optimization was not the primary concern. Swift is not even feature-complete! They will make it better.

Allocation(heap/stack)
Reference Counting(reatain release cost times)
Dispatch(inline\static\dynamic) and objects(instance RC operation)
Structs(heap)
Abstactions
 
 [Real World Swift Performance](https://realm.io/news/real-world-swift-performance/)
 
 [Writing High-Performance Swift Code](https://github.com/apple/swift/blob/master/docs/OptimizationTips.rst)