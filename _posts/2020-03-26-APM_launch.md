
---
title:  APM
header: APM
description: APM
---

1. app 启动优化
[apple 官方文档](https://developer.apple.com/documentation/xcode/improving_your_app_s_performance/reducing_your_app_s_launch_time?language=objc)

动态库加载：加载app执行文件、测试mach加载命令去寻找app需要的framwork和动态库。

ps:动态库和静态库的区别，静态库文件类型有：.a 和 .framework, 动态库文件类型有.dylib和.framework。 静态库是指在编译时拷贝到目标程序中，编译后可以脱离源码存在，如果静态库中有代码修改，需要重新编译拷贝。而动态库则是在目标程序运行前，将动态库和目标程序绑定，多个程序可共享一个动态库。

cocoapods 中使用 use_framworks! 就是使用动态库的方式。

静态初始化：这些代码必须在app的main函数之前。包括：
C++静态构造器。

OC分类中定义的load方法。

通过clang属性标记的__attribute__(constructor)方法。

app中或framework二进制文件中链接到__DATA和__mod_init_func部分的函数 。


UIKit 声明周期的一些方法：
UIKit初始化app的delegate的实例然后发送 application:willFinishLaunchingWithOptions:和 application:didFinishLaunchingWithOptions:消息。这是在主线程发送的，所以延迟一些非必要的任务到之后的一段合适的时间来做。

画初始view的识图层级：
简化初始化view的复杂度，用标准视图替换掉重写了drawRect:的自定义视图

    class ViewController: UIViewController {
        static let startupActivities:StaticString = "Startup Activities"
        let poiLog = OSLog(subsystem: "com.example.CocoaPictures", category: .pointsOfInterest)

    override func viewDidLoad() {
        super.viewDidLoad()
        os_signpost(.begin, log: self.poiLog, name: ViewController.startupActivities)
        // do work to prepare the view
        os_signpost(.end, log: self.poiLog, name: ViewController.startupActivities)
        }   

在scheme中加入DYLD_PRINT_STATISTICS 设置为1
更详细的使用DYLD_PRINT_STATISTICS_DETAILS 设置为1
DYLD_PRINT_LIBRARIES 打印加载顺序

    Total pre-main time: 131.80 milliseconds (100.0%)
             dylib loading time:  45.79 milliseconds (34.7%)
            rebase/binding time: 411015771.6 seconds (336794286.5%)
                ObjC setup time:  13.86 milliseconds (10.5%)
               initializer time:  73.53 milliseconds (55.7%)
               slowest intializers :
                 libSystem.B.dylib :   7.22 milliseconds (5.4%)
       libBacktraceRecording.dylib :  13.29 milliseconds (10.0%)
        libMainThreadChecker.dylib :  35.93 milliseconds (27.2%)
      libViewDebuggerSupport.dylib :  10.59 milliseconds (8.0%)

      total time: 1.0 seconds (100.0%)
      total images loaded:  384 (378 from dyld shared cache)
      total segments mapped: 21, into 400 pages
      total images loading time: 571.10 milliseconds (52.1%)
      total load time in ObjC:  13.86 milliseconds (1.2%)
      total debugger pause time: 525.31 milliseconds (48.0%)
      total dtrace DOF registration time:   0.00 milliseconds (0.0%)
      total rebase fixups:  17,959
      total rebase fixups time:  24.00 milliseconds (2.1%)
      total binding fixups: 523,279
      total binding fixups time: 411.76 milliseconds (37.6%)
      total weak binding fixups time:   0.05 milliseconds (0.0%)
      total redo shared cached bindings time: 437.21 milliseconds (39.9%)
      total bindings lazily fixed up: 0 of 0
      total time in initializers and ObjC +load:  73.53 milliseconds (6.7%)
                             libSystem.B.dylib :   7.22 milliseconds (0.6%)
                   libBacktraceRecording.dylib :  13.29 milliseconds (1.2%)
                               libobjc.A.dylib :   2.10 milliseconds (0.1%)
                    libMainThreadChecker.dylib :  35.93 milliseconds (3.2%)
                  libViewDebuggerSupport.dylib :  10.59 milliseconds (0.9%)
    total symbol trie searches:    1301730
    total symbol table binary searches:    0
    total images defining weak symbols:  41
    total images using weak symbols:  105

这里我们可以看到分别为 动态库加载时间，绑定时间（动态链接库在虚拟内存中的地址与程序的绑定），OCsetup的时间，初始化的时间。

总耗时里可以看到 镜像加载的时间，segments 映射的时间， 镜像加载时间，Objc 加载时间，debug 暂停时间，rebase修复个数和时间，绑定修复个数和时间，弱绑定时间，共享缓存重做绑定时间，懒加载绑定修复，符号查找，符号表二进制查找，定义若符号镜像，使用弱符号镜像。


在每个动态库的加载过程中， dyld需要：

分析所依赖的动态库

找到动态库的mach-o文件

打开文件

验证文件

在系统核心注册文件签名

对动态库的每一个segment调用mmap()



针对这一步骤的优化有：

减少非系统库的依赖；

使用静态库而不是动态库；

合并非系统动态库为一个动态库；

[比较全的启动优化博客](https://blog.csdn.net/olsQ93038o99S/article/details/81518485)

[与上个一毛一样](http://www.cocoachina.com/articles/24423)


[optimizing App Startup Time-WWDC2016](https://developer.apple.com/videos/play/wwdc2016/406/?time=1536)
[optimizing App Startup Time-WWDC2016](https://developer.apple.com/videos/play/wwdc2016/406/?time=1536)
[optimizing App Startup Time-WWDC2016](https://developer.apple.com/videos/play/wwdc2016/406/?time=1536)


WWDC的视频分为两个大部分，第一个部分是了解启动时发生了什么，第二部分是如何去优化。
第一部分有四个理论知识:
main() 函数之前的做了什么，Mach-O格式，虚拟内存基础，Mach-O是怎么加载和准备的。
第二部分：
如何去测量，怎么去优化。

首先一些速成课。
什么是Mach-O: 一组不同的运行时可执行文件的文件类型。

第一个是可执行文件：他是应用的主要二进制文件。
第二个是dylib： 他是一个动态库(dynamic library)，你也许讲过DSOs或者DLLs。
第三个是Bundle：bundle是一种特殊的动态库，在iOS或者说apple的平台上是一种不能直接链接的动态库，只能在运行时通过一个dlopen()来加载，dlopen是mac-os的一个插件。

这三种类型全部是镜像的一种。然后我们常常看到的framework是一种特殊的dylib,它使用特殊的目录结构来持有这个ddylib需要的文件。


一个Mach-O镜像被分成了段（segment），名字通常大写，并且每个segment都有页数，页数的大小是由硬件决定的，arm64上是16kb，其他是4kb。
接下来是分区（section），分区会被编译器忽视，每个section可以任意大小，不必遵守页大小的限制。

常见的段（segment）：TEXT、DATA、LINKEDIT，几乎每个二进制文件都有这些段。可以自定义段，但是不能加任何值。TEXT是Mach-O文件的开始，里包含mach的header、机器指令和一些只读的常亮（比如c字符串），是可读可执行的（r-x）。DATA里放的是以全局变量和静态变量，是可读写的（rw）。现在LINKEDIT里没有了函数名和全局变量（那么以前可能是放在合格位置的），现在里面有函数的变量和地址。是只读的（r）。通用文件是包含了多个架构的Mach-O的文件，比如把arm64的Mach-O和armv7s的Mach-O放到合并到另一个文件中，这个文件也有一个头（Fat Header），里面包含了架构的列表，以及每个架构的在文件内的偏移量。这个文件头也有占一个页的大小(16K或4K)。为什么一个头要占一页的大小，并且一个segment要占多页的大小，这有些浪费空间。这就要说到虚拟内存（virtual memory）。软件工程师中有句格言，任何事都可以通过添加一个间接层来解决。当所有这些进程存在时，如何管理这些物理内存，这就是虚拟内存要解决的问题，通过添加一个间接层。

每个进程的逻辑地址都会映射到物理内存的地址，这个映射不必是一对一的，可能映射不到实际的物理内存，也有可能多个逻辑地址映射到一个物理内存上。那么通过VM我们可以做什么呢，如果进程内的逻辑地址映射到一个不存在的物理地址，那么这是就会发生页错误(page fault)，然后内核要去停止线程去搞清楚他接下来要做什么，如果有两个进程的内存是不一样的，但是映射到了同一个物理内存上，这两个进程就共享了物理内存。

还有个有趣的特点叫做文件支持的映射，当进程要是某个文件时，没有必要把整个文件全部加载到内存中，可以通过VM的mmap来加载文件的一个范围，当我们第一次访问从未访问过的地址时，内核就会去文件读一个相应的page。这就可以去懒读取文件，把上面的所有特性融合到一起，如果有多个进程映射到一个dylib或image的TEXT段，他就会被懒读取，并且TEXT中的这些也被多个进程共享。那DATA段，他是可读写的，有一个东西叫COW(copy-on-write)。一开始是多个进程间共享DATA页，他们只读取DATA页中的数据，一旦一个进程发生了写操作改变了数据，拷贝操作就会发生。然后h要重定向进程的地址与物理内存。

这就引发了clean and dirty page，脏页有着进程的独特的信息，干净页可以从物理m内存中重新读取，所以脏页的代价要更高。

最后一点是，我们可以标记一个页的权限为可读，可写，可运行，或者这些的结合，上面介绍了Mach-O和VM，接下来把这些综合起来。

先看一下dyld是如何工作的，我们有一个dylib文件，我们没有把他加载到内存中，而是把它映射到内存中，假如有三页的TEXT，一页的DATA和LINKEDIT，所以在内存中有八页的大小，大部分全局变量都会被初始化为0，静态链接器做一个优化，把所有为0的全局变量移动到后面，这样就不会占据磁盘空间了，然后我们会让VM第一次访问这一页时用0去填充，就不需要去读了。dyld做的第一件事是去读mach header，在这个进程的内存的顶端，什么都没有，这时候就发生了页错误，然后内核意识到这是一个映射，他就会去读取这个文件的第一个page，放到物理内存上去，设置映射，然后dyld可以读取到文件的头，然后mach header 告诉dyld你还要去读取我的LINKEDIT，同样去内存的底部发现什么也没有，然后就发生了页错误，再去文件读取一页，映射到内存。然后LINKEDIT告诉dyld你需要去做一些DATA的修复来保证这个dylib可运行。然后同样的事发生了，dyld去读DATA的那一页，这里有一些不同的地方，dyld是要写回一些数据的，需要改变DATA页。然后COW就会运行，这一页就变成了dirty page。如果我开辟了八页的空间，然后读取了整个文件的内容，然后这八页都是dirty page，但是我现在只有一页dirty和两页clean。

当第二个进程加载这个dylib时也是一样，先去看看mach-header，这时候内核告诉进程已经有一个在内存里了，只需要映射一下就好了，LINKEDIT也是一样，然后是DATA，这时候去是否有clean page的DATA，如果有直接映射，如果没有就重读一个到新的内存中。现在在这个进程中dyld会让RAM变脏。现在最后一步是只有当dyld需要操作时才会调用LINKEDIT，所以它可以提醒内核，现在我不需要LINKEDIT了，你可以回收这些页如果别的RAM需要时。

现在有两个进程共享这个dylib，每个本来应该有8个页的总共16个脏页，但是我们只有两个dirty page和一个共享的clean page。另外我想说明的两点是安全怎么影响dyld的。一个是ASLR(Address Space Layout Random)，基本概念是把加载地址随机化, 另一个是签名，对整个文件使用加密hash算法，然后在文件上签名。

所以为了在运行时验证，意味着整个文件要被重读一次。所以，取而代之的是在编译时把每个页独立hash，然后存到LINKEDIT中。这就使得每个未被修改的页在读取时都能够及时的验证。

这就是速成课的所有内容。接下来就要从exec介绍到main。什么exec，exec是一个系统调用，当你进入内核，告诉内核我要替换这个进程为一个新的程序，内核会抹去整个地址空间并且映射到你指定的运行文件，ASLR把它映射到一个随机地址。下一步从该随机地址回溯到0，标记整个区域为不可访问的，也就是不可读不可写不可运行。这个区域在32位上至少为4KB在64位上至少为4GB。※※※※会捕获任何空指针以及任何指针截断※※※※

UNIX诞生的前几十年都很简单，我只需映射一个程序，把指正引用指向它，然后开始运行。然后发明了共享库，那么谁来加载dylib呢，人们很快意识到情况太过复杂，不想让内核来做这件事，所以就创建了一个帮助程序，在apple的平台叫做dyld。在其他的Unix平台你也许知道LD.SO 所以当内核完成映射一个进程还会映射另外一个叫做dyld的Mach-O到进程中的随机地址，把指针引用指向dyld，然后让dyld来完成加载进程。所以现在dyld运行在进程中，他的任务是加载你依赖的所有dylibs，让他们全部准备好开始运行。

我们一起来浏览这些步骤，在底部有很多步骤和时间线，我们浏览这些的时候也会浏览时间线。dyld要映射依赖的dylibs。有哪些dylib呢？首先去读取主运行文件的header，这个header是一个依赖的dylid的表，内核早映射好了的。这需要解析。必须找到每个dylib。一旦找到每个dylib就会打开并且运行每个文件，确保是一个正确的Mach-O文件，验证，找到他的签名，并且注册到内核中。然后它可以在dylib的每一个段里调用mmap。目前还挺简单的，dyld加载了应用需要的dylib，比如A和B，那么A和B还依赖了其他的bylib呢，每个dylib可能依赖了其他的已加载或未加载的dylib，知道所有的都加载完毕，回过头来看一个进程，在系统中平均一个进程需要100-400个dylib，这有很多。幸运的是大部分都是OS的dylibs，在OS建立时会预计算和预缓存了很多dyld在加载时要做的事。所以OS dylib加载的非常快。现在我们完成了加载这些lib，但是他们之间都是独立存在的，现在我们需要把他们绑定到一起，这叫做fix-up。

因为代码签名的存在，我们无法修改dylib的指令，如果不能改变调用的指令那么一个dylib如何调用另一个dylib的呢，这时候需要添加很多的老的间接层。※※※※所以我们的code-gen，一个被叫做dynamic PIC的东西※※※※


slide = 内存地址-真实地址。这就需要rebasing和bind，rebasing是Mach-Oe内部在使用是需要知道自己的数据在什么位置，bind是保证正确访问外部的地址。再有TEXT和DATA都是有hash之后加上签名，然后把这些数据放在LINKEDIT中，每次使用是都要先到ILINKEDIT中去签名然后验证是否一致。

借助于虚拟内存技术，应用在运行时使用的内存地址和实际的物理内存地址不是一一对应的，如果使用的内存地址在实际的物理内存不存在，内核就会报错误。

系统的大部分库都会在系统启动时提前算好，所以在dyld时几乎不耗时间，这里的大部分时间都是应用内存自己加入的一些动态库。我们有时候会看到，app连续两次启动耗时差别在dyld时比较大，这是因为耗时大的那部分的动态库被缓存了，所以不需要再重新dyld。最好的测量方式是重新启动设备，然后启动app测量。

启动时间的标准，就是控制在400ms左右，包括finishLaunchWithOptions:的时间

在scheme的环境变量中 添加DYLD_PRINT_STATISTICS

然后启动前分五个阶段

dyld-> rebase-> bind -> objc setup-> initializer

dyld 这里呢就是尽量减少动态库的数量，或者把所有动态库放到静态库里，还有额外的建议把cocoapods的use_framwork! 注释掉
rebase 和bind的操作就是前者所做的工作。
objc setup 这部分就是 把oc的类名和属性放到TEXT中，所以我们要做的就是减少类的数量，1k到2k都是可以的。如果到5k，1w，2w，那么就要考虑要不要那么多的类了，尤其是一些类中之后一两个属性 和一两个方法。
最后是initialzier ：
这部分分为显性和隐性：
显式的：类的load和类别的loadj，建议放到initializer方法中，还有C++的 attribute的constructor。

隐式：C++的虚函数。

建议用swift实现，因为swift的消耗要很小，至于为什么就没说了。
