
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

我们一起来浏览这些步骤，在底部有很多步骤和时间线，我们浏览这些的时候也会浏览时间线。dyld要映射依赖的dylibs。有哪些dylib呢？首先去读取主运行文件的header，这个header是一个依赖的dylid的表，内核早映射好了的。这需要解析。必须找到每个dylib。一旦找到每个dylib就会打开并且运行每个文件，确保是一个正确的Mach-O文件，验证，找到他的签名，并且注册到内核中。然后它可以在dylib的每一个段里调用mmap。目前还挺简单的，dyld加载了应用需要的dylib，比如A和B，那么A和B还依赖了其他的dylib呢，每个dylib可能依赖了其他的已加载或未加载的dylib，知道所有的都加载完毕，回过头来看一个进程，在系统中平均一个进程需要100-400个dylib，这有很多。幸运的是大部分都是OS的dylibs，在OS建立时会预计算和预缓存了很多dyld在加载时要做的事。所以OS dylib加载的非常快。现在我们完成了加载这些lib，但是他们之间都是独立存在的，现在我们需要把他们绑定到一起，这叫做fix-up。

因为代码签名的存在，我们无法修改dylib的指令，如果不能改变调用的指令那么一个dylib如何调用另一个dylib的呢，这时候需要添加很多的老的间接层。※※※※所以我们的code-gen，一个被叫做dynamic PIC的东西，定位到独立代码，即代码可以动态的加载到z地址中，也就是说地址被间接的分配，也就是说当一个指令调用另一个指令，co-gen实际上是a在DATA段中创建了一个指针，这个指针指向要调用的地方。代码加载指针也跳向该指针。所以dyld所做的就是修复这些指针和数据。※※※※

修复有两种，重设基址和绑定，重设基址是指在镜像内的指针调整，绑定是指指向镜像外的指针。这两种方式需要不同的修复。有一个命令dyldinfo，它有一些参数选项，可以在任何二进制上运行，可以看到dyld在这个准备这个二进制所准备的工作。

    xcrun dyldinfo -rebase -bind -lazy_bind myapp.app/myapp
    
通过这个命令看到的信息大部分是一些动态库里的方法名，比如libobjc里的_objc_开始的一些方法，sqlite3里的一些C方法名字等等。

在以前可以指定一个首选加载地址来加载dylib，这个地址是静态链接的和dyld一起工作，当dylib加载到z首选地址后，所有指向代码内部的指针都被正确内联，不需要啊任何修正。但是现在由于ASLR，dylib会被加载到随机的地址。会被滑动到其他地址去，但是现在DATA的指针全是指向老的位置，为了修复这些，所以需要计算滑动后的地址，也就是它移动了多远，对每一个内部指针都要加上滑动的距离。总的来说重设机制就是把所有内部指针都s加上一个滑动值，操作就是读，加，写，读，加，写。这些数据指针被编码后在LINKEDIT段里，这时所有的映射都完成了。当我们开始重设基址时，所有的DATA页都会产生页错误，然后当我们尝试修改的时候就会发生COW。所以有时候因为IO的原因重设基址有很大的代价，有个小技巧，从内核的角度看我们是有序的读取，然后内核看到数据错误也是有序的，内核就会提前读取使IO操作的消耗更小。

下面是绑定，绑定是用于指向dylib的外部的指针，以名字来区分，名字实际上是一个字符串。在ppt中的例子，malloc是存在LINKEDIT中的，也就是说这个指针需要指向malloc。所以在运行时，dyld需要发现这个符号的实现，需要去遍历符号表，这需要一个大量的计算，就会存到DATA的指针里，这个比重设基址需要更复杂的计算。但是在绑定阶段需要很少的IO，因为在重设基址阶段啊几乎已经完成了所有的IO。下一步Objc 有很多DATA 结构，DATAl结构类是指向它的方法和高光的指针（super gloss）。

到这里通过重设基址和绑定，所有的修复几乎就都完成了。但是在OC的运行时需要做一些额外的事情，因为OC是动态语言，所有OC可以通过名字的方式来实例化(substantitated)类。

在OC的运行时需要维护一张包含所有类的名字的映射的表，每次加载的名称都定义了一个类，它的名字需要注册到一个全局表中。相信你们在C++都听过易碎的基类的问题，这个问题在OC中通过动态的改变IVar的偏移量来解决了，我们可以在OC中定义分类来改变一些类的方法。有时候一些分类类不是在dylib的镜像里的而是在定义类的地方，所以这个时候需要完成类别里的方法的修复。

最后，Objc是基于selector的，所以selector必须是唯一的。现在所有的DATA的修复都完成了，目前所说的DATA修复都是静态的，下面是动态的修复的时机了，在C++中，有一个初始化器，你可以定义任何你想要的表达式。需要在这个时候被运行的任意表达式，所以编译器为这些任意的DATA初始化生成了初始化器。

在Objc中有个一个load方法，****load方法被废弃了，建议不要用（对此持一问态度）****，最好使用initializer类方法，如果你使用了load方法，那么这时候就会被调用了。现在有一张大图，顶端是所有的dylib依赖的主要的可执行文件都在这张大图里，我们需要运行初始化器。它是从下往上运行的。原因是，当我们运行初始化时，它也许会调用到一些别的dylib，然后要确保这些dylibu需要已经准备好被调用了。所以我们从底部运行初始化器知道app的类所有调用的东西都是安全的。一旦所有的初始化器完成了，现在终于可以调用dyld的主程序了。

在不同的设备上启动时间是不同的，一个良好经验：400ms是一个很好启动时间，因为在主屏幕上和应用之间有一个启动动画给与了一种持续感，这个动画时间就可隐藏了启动时间。应用扩展程序也是应用启动的一部分。但是不应该超过20s，系统会杀掉app的。系统以为进入了一个死循环。使用最差的设备来测试，因为你在好的设备上测试可能能达到，但是一旦在更差的设备上可能就达不到了。

我们在启动时需要做什么，解析镜像，映射镜像，rebase镜像，bind镜像，运行镜像初始化器，然后调用main方法，在这之后又会调用UIApplicationMain方法，在OC和swift里都可以看到他的隐式调用，它还会做一些其他操作，包括运行framwork的初始化器和加载nib文件。然后会运行UIApplication的Delegate的回调application:DidFinishLaunchWithOptions:。

冷启动与热启动。热启动是app已经在内存中，因为它之前启动过然后退出了，因为它还存在内核的硬盘缓存中(discache)，或者因为你只是复制了它。冷启动是重要的测量依据，因为用户重启设备后，第一次启动可能会比较耗时，但是你想立刻加载。所以为了准确测试需要在机器重启的情况下。通过DYLD_PRINT_STATISTICS。为了从你的app中解析符号和加载你的断点，所以debugger在每一个dylib启动时通过USB线的暂停是非常耗时的。但是dyld知道这些时间，会从启动时间中减去。


嵌入的动态库是非常耗时的。平均一个app会用到100到400个动态库，但是系统动态库是很快的，因为我们提前计算了很多数据。我们无法提前计算你app中嵌入的一些别的动态库，所以这个加载起来非常慢。所以就是减少库的数量，或者用动态打包的方式来链接这些库。也可以去懒加载，使用dlopen命令，但是dlopen会引起一些性能和正确性的问题，需要以后u做更多的操作，但是这些操作确实被延迟了。

重设基址需要IO操作，绑定需要计算，重设基址由于IO操作会更慢一些，但是绑定的IO是在rebase中做完了的，所以他们的时间是叠加在一起的。这些s修复都是DATA部分的修复。所以我们必须少修复一些指针数量，dyldinfo可以看到这里面有哪些段和部分。比如你看到一个符号在Objc部分的Objc类，所以可能有很多的OC的类，所以我们可以减少OC的类和变量的数量。另外一个可以尝试的方法是减少xC++的虚函数的数量，虚函数创建了一个叫V表的东西，和Objc的元数据是一样的，都创建了DATA 结构的，都是需要被修复的，虚函数比Objc和Objc的元数据小，但是对于一些应用来说作用是很明显的，可以使用swift的结构体，swift的结构使用更少的需要被修复的指针，同时swift的结构体也更加的内联，通过更好的co-gen来避免许多操作。还要小心机器生成的代码，有个例子，通过DSL或者自定义的语言来生成结构体，然后通过应用程序来生成代码。如果这个生成代码的程序有许多指针，当你生辰非常非常大的结构体时是非常g昂贵的，产生了兆量级的数据。但是如果使用其他的方式而不是指正的方式能够控制很多的东西，比如偏移基址的结构体。

有两种初始化器，显式初始化器，比如load用initializer代替，C++中函数可以带有一个属性，可以导致函数像初始化器一样生成代码，这是显式初始化器，可以用dispatch_once,pthred_once,std::once是替换。

隐式初始化器。大部分来自C++的有※※※※非平凡(non-trival)※※※※的初始化器全局变量。可以使用推荐的初始化器（site initializer）替换，当然有些地方，可以放置非全局结构体的变量或者指向要初始化的对象的指针。还有一个选项，是不使用非平凡初始化构造器。在CC++中有一个叫做POD的初始化构造器，一个普通的旧数据※※※※。如果你的对象仅仅是静态的普通旧数据，或者静态链接器会提前计算DATA部分的数据。把它作为普通数据显示出来，不需要运行，不需要修复。最后一步非常难发现，因为他是隐藏的，但是在编译器中有一个配置-Wglobal-constructors，如果你使用了它，它会在你生成隐式初始化器的时候给你警告。


还有一个选项就是用swift重新编写，swift有会被初始化的全局变量，他们会保证在使用之前会被初始化，但是不是用初始化器来实现的，在幕后使用了dispatch_once。使用了一个叫做调用点初始化器的东西。

在初始化器中不要调用dlopen，它将带来巨大的性能问题，当dyld在app启动前运行，我们可以关闭锁，因为我们是单线程的。当dlopen调用了，初始化器的运行就发生了改变，将会有多线程的问题，就必须打开锁，将会使性能发生巨大的下降。还会带来细微的死锁和未定义的行为。同样基于童颜的原因不要在初始化器中开启线程，如果必须的话可以建立一个mutex，

You also can have subtle deadlocking and undefined behaviors. Also, please don't start threads in your initializers, basically for the same reason. You can set up a mute text if you have to and mute text even have like, preferred mute texts even have, predefined static values that you can set them up with that run no code. But actually starting a thread in your initializer is, potentially a big performance and correctness issue. So here we have some code, I have a C++ class with a non-trivial initializer.




