---
title: The interview questions summary
description: This is an introduction to summary of interview questions
header: today interveiew summary
---
Rules are meant to be broken
There is a kind of beatuy in imperfection
If you love life, life will love back.

今日面试总结：
1. cocoapods原理

Pods-ProjectName-frameworks.sh脚本负责在每次编译的时候，帮你把预引入的所有三方库文件打包的成ProjectName.a静态库文件，放在我们原Xcode工程中Framework文件夹下，供工程使用。

Resource资源文件主要由Pods工程中的Pods-ProjectName-resources.sh脚本负责，在每次编译的时候，该脚本会帮你将所有三方库的Resource文件copy到目标目录中。

在Pods工程中的的每个库文件都有一个相应的SDKName.xcconfig，在编译时，CocoaPods就是通过这些文件来设置所有的依赖参数的，编译后，在主工程的Pods文件夹下会生成两个配置文件，Pods-ProjectName.debug.xcconfig、Pods-ProjectName.release.xcconfig。

1、pod install提速

每次执行pod install和pod update的时候，cocoapods都会默认更新一次spec仓库。这是一个比较耗时的操作。在确认spec版本库不需要更新时，给这两个命令加一个参数跳过spec版本库更新,可以明显提高这两个命令的执行速度。

pod install --verbose --no-repo-update
pod update --verbose --no-repo-update
2、关于Podfile文件编辑时，第三方库版本号的各种写法

pod ‘AFNetworking’ //不显式指定依赖库版本，表示每次都获取最新版本
pod ‘AFNetworking’, ‘2.0’ //只使用2.0版本
pod ‘AFNetworking’, ‘>2.0′ //使用高于2.0的版本
pod ‘AFNetworking’, ‘>=2.0′ //使用大于或等于2.0的版本
pod ‘AFNetworking’, ‘<2.0′ //使用小于2.0的版本
pod ‘AFNetworking’, ‘<=2.0′ //使用小于或等于2.0的版本
pod ‘AFNetworking’, ‘~>0.1.2′ //使用大于等于0.1.2但小于0.2的版本，相当于>=0.1.2并且<0.2.0
pod ‘AFNetworking’, ‘~>0.1′ //使用大于等于0.1但小于1.0的版本
pod ‘AFNetworking’, ‘~>0′ //高于0的版本，写这个限制和什么都不写是一个效果，都表示使用最新版本

如果说 CocoaPods 像一个航母, 一应俱全, 坚实稳固. 那么 Carthage 就像一艘巡洋舰, 机动灵活, 攻击迅速. 1
Why to use Carthage? 2

CocoaPods是已存在很长时间的Cocoa依赖管理器, 那么为什么要创建Carthage呢?
1) CoaoaPods 是一套整体解决方案，我们在 Podfile 中指定好我们需要的第三方库。然后 CocoaPods 就会进行下载，集成，然后修改或者创建我们项目的 workspace 文件，这一系列整体操作。
2) 相比之下，Carthage 就要轻量很多，它也会一个叫做 Cartfile 描述文件，但 Carthage 不会对我们的项目结构进行任何修改，更不多创建 workspace。它只是根据我们描述文件中配置的第三方库，将他们下载到本地，然后用 xcodebuild 构建成 framework 文件。然后由我们自己将这些库集成到项目中。Carthage 使用的是一种非侵入性的哲学。

Carthage 基本的工作流程：
1> 创建一个Cartfile，包含你希望在项目中使用的框架的列表
2> 运行Carthage，将会获取列出的框架并编译它们
3> 将编译完成的.framework二进制文件拖拽到你的Xcode项目当中
Carthage编译你的框架/库，然后提供该框架的二进制文件，但你仍然持有该项目结构和设置的绝对控制。Carthage不会自动的修改你的项目文件或编译设置。

相信大家可能遇到这种情况, Podfile中配置好相关框架/库 -> pod install -verbose -no-repo-update, 然后编译运行时, 出现类似错误:

diff: /../Podfile.lock: No such file or directory
diff: /Manifest.lock: No such file or directory

接下来又是一系列的折腾, 白白浪费很多时间.
Carthage or CocoaPods? 3

CocoaPods 有如下优势:

    ① 使用方便, 除编写 Podfile 以外其他几乎都是自动完成;
    ② 软件包数量多，主流支持；
    ③ 支持 iOS 8 Framework，当然也支持旧的静态编译.

但是 CocoaPods 作为一个有中心仓库的解决方案，缺点也比较明显：

    1?? 每次更新环境都需要连接到中心仓库，比较耗时；
    2?? 开发者使用比较简单，但是如果创建兼容 CocoaPods 的库，就会相对繁琐一些（尽管有了命令行）；
    3?? 每次干净编译都会把所有第三方库都重新编译一次

Carthage 的优势:

    ① 使用 Carthage 的话，所有的第三方库依赖，除非是更新的需要，不然平常干净编译 Project，它是不需要再次编译的，大大加快平常编译及 Archive 的时间.
    ② 它是去中心化的，没有中心服务器. 这意味着每次配置和更新环境，只会去更新具体的库，而不会有一个向中心服务器获取最新库的索引这么个过程，如此又省了很多时间.
    ③  CocoaPods 无缝集成！一个项目可同时使用两套包管理工具, 当前 CocoaPods 管理主要 Framework 的配置下, 将少量其他 Framework 交给了 Carthage 管理, 二者可以和谐地共存.
    ④ 结构标准的项目天然就是 Carthage 库.

Carthage 的不足：

    1?? 库依然不如 CocoaPods 丰富：尽管很多库不需要声明并改造就直接可以被 Carthage 用，但依然有大量 CocoaPods 能用的库不支持，我相信时间能解决这个问题；
    2?? 只支持 Framework，所以是 iOS 8 Only了，随着时间推移，这个也不会是问题；
    3?? 工具仍不完善：在使用过程中，发现它无法在一个结构复杂的项目中正确发现库（比如有 iOS, Mac demo + framework 的结构）；
    4?? 无法在 Xcode 里定位到源码：如果你在写代码过程中，想跳转到一个第三方库去看具体的实现，这是无法办到的，Carthage 的配置只能让你看到一个库的头文件

Installing Carthage 开始使用：4

1. 创建一个"Cartfile"，将你想要使用的框架列在里面2. 运行"carthage update"，将获取依赖文件到一个Carthage.checkout 文件夹，然后编译每个依赖3. 在你的应用程序target的 'General' 设置标签中的 'Embedded Binaries' 区域，将框架从"Carthage.build文件夹拖拽进去"。

在这个过程当中，Carthage将创建一些build artifacts，其中最重要的是Cartfile.lock
文件，里面将列出每个框架的具体版本，确保你提交了这个文件到版本控制工具里面（如Git、SVN），因为每个用到项目的人都需要它来编译相同版本的框架。
完成上面的步骤并提交你的修改，项目的其他用户就只需要获取该仓库并执行carthage bootstrap 就能使用你所添加的框架。

添加框架到单元测试或另一个框架
使用Carthage添加框架到任意目标的方法，和添加到应用程序差不多。主要的不同在于框架是如何设置并链接到Xcode的。
因为非应用程序目标没有“Embedded Binaries”设置区域，你需要将编译完成后的框架拖拽到“Link Binaries With Libraries”的区域里。
在某些稀有案例中，你也许会想要复制每个依赖到已编译的项目中（比如，在外部框架中嵌入依赖，或确保依赖在测试工具中正常显示）。想要达到这个目的，你需要创建一个新的“Copy Files”编译选项和“Frameworks”组，然后将框架的引用添加到里面。

升级框架
如果你修改了Cartfile，或者你想升级到框架的最新版本（取决于你指定的需求版本），执行 carthage update 命令可以达到目的。

让你的框架支持Carthage
Carthage只正式支持动态框架，动态框架能够在任何版本的OS X上使用，但只能在iOS 8及以上版本使用。
因为Carthage拥有非中心化的包列表，以及没有项目指定的编译设置，大多数框架应该能自动编译。

分享你的Xcode schemes
Carthage将只从你的.xcodeproj 中标记为已分享的Xcode schemes来编译。如果你想检查编译是否成功，执行carthage build --no-skip-current命令，然后检查Carthage.build文件夹。
如果当执行命令但有scheme没有被编译，打开Xcode并确定对应scheme被标记为“Shared”，以便Carthage能够发现它。

解决编译失败
如果你在执行carthage build --no-skip-current
时编译失败，尝试执行xcodebuild -scheme SCHEME -workspace WORKSPACE build 或 xcodebuild -scheme SCHEME -project PROJECT build（将其中的大写单词换成你项目的对应名称），然后观察是否有相同的失败发生，这应该能生成足够的失败信息来解决问题。

基本的工作流如下：

    创建一个Cartfile，包含你希望在项目中使用的框架的列表

    运行Carthage，将会获取列出的框架并编译它们

    将编译完成的.framework二进制文件拖拽到你的Xcode项目当中

Carthage编译你的依赖，并提供框架的二进制文件，但你仍然保留对项目的结构和设置的完整控制。Carthage不会自动的修改你的项目文件或编译设置。
Carthage与CocoaPods的不同

CocoaPods是已存在很长时间的Cocoa依赖管理器，那么为什么要创建Carthage呢？

首先，CocoaPods默认会自动创建并更新你的应用程序和所有依赖的Xcode workspace。Carthage使用xcodebuild来编译框架的二进制文件，但如何集成它们将交由用户自己判断。CocoaPods的方法更易于使用，但Carthage更灵活并且是非侵入性的。

CocoaPods的目标在它的README文件描述如下：

    …为提高第三方开源库的可见性和参与度，创建一个更中心化的生态系统。

与之对照，Carthage创建的是去中心化的依赖管理器。它没有总项目的列表，这能够减少维护工作并且避免任何中心化带来的问题（如中央服务器宕机）。不过，这样也有一些缺点，就是项目的发现将更困难，用户将依赖于Github的趋势页面或者类似的代码库来寻找项目。

CocoaPods项目同时还必须包含一个podspec文件，里面是项目的一些元数据，以及确定项目的编译方式。Carthage使用xcodebuild来编译依赖，而不是将他们集成进一个workspace，因此无需类似的设定文件。不过依赖需要包含自己的Xcode工程文件来描述如何编译。

最后，我们创建Carthage的原因是想要一种尽可能简单的工具——一个只关心本职工作的依赖管理器，而不是取代部分Xcode的功能，或者需要 让框架作者做一些额外的工作。CocoaPods提供的一些特性很棒，但由于附加的复杂性，它们将不会被包含在Carthage当中。
安装Carthage

Carthage提供OS X平台的pkg安装文件，你可以从Github的最新release中找到，按照引导一步步安装即可。

如果你想安装最新的开发版本（可能存在稳定性和兼容性的问题），你只需要clone本仓库的master分支，然后运行make install.
添加框架到应用程序

安装完Carthage后，你能够使用它来添加框架到你的项目。注意Carthage只支持动态框架，而后者只存在于iOS 8以上（以及任意版本的OS X）。

开始使用：

    创建一个Cartfile，将你想要使用的框架列在里面

    运行carthage update，将获取依赖文件到一个Carthage.checkout文件夹，然后编译每个依赖

    在你的应用程序target的“General”设置标签中的“Embedded Binaries”区域，将框架从Carthage.build文件夹拖拽进去。

在这个过程当中，Carthage将创建一些build artifacts，其中最重要的是Cartfile.lock文件，里面将列出每个框架的具体版本，确保你提交了这个文件到版本控制工具里面（如Git、SVN），因为每个用到项目的人都需要它来编译相同版本的框架。

完成上面的步骤并提交你的修改，项目的其他用户就只需要获取该仓库并执行carthage bootstrap就能使用你所添加的框架。
添加框架到单元测试或另一个框架

使用Carthage添加框架到任意目标的方法，和添加到应用程序差不多。主要的不同在于框架是如何设置并链接到Xcode的。

因为非应用程序目标没有“Embedded Binaries”设置区域，你需要将编译完成后的框架拖拽到“Link Binaries With Libraries”的区域里。

在某些稀有案例中，你也许会想要复制每个依赖到已编译的项目中（比如，在外部框架中嵌入依赖，或确保依赖在测试工具中正常显示）。想要达到这个目的，你需要创建一个新的“Copy Files”编译选项和“Frameworks”组，然后将框架的引用添加到里面。
升级框架

如果你改动了你的Cartfile，或者你想升级到框架的最新版本（服从于你指定的需求版本），执行carthage update命令可以达到目的。
让你的框架支持Carthage

Carthage只正式支持动态框架，动态框架能够在任何版本的OS X上使用，但只能在iOS 8及以上版本使用。

因为Carthage拥有非中心化的包列表，以及没有项目指定的编译设置，大多数框架应该能自动编译。
分享你的Xcode schemes

Carthage将只从你的.xcodeproj中标记为已分享的Xcode schemes来编译。如果你想检查编译是否成功，执行carthage build --no-skip-current命令，然后检查Carthage.build文件夹。

如果当执行命令但有scheme没有被编译，打开Xcode并确定对应scheme被标记为“Shared”，以便Carthage能够发现它。
解决编译失败

如果你在执行carthage build --no-skip-current时编译失败，尝试执行xcodebuild -scheme SCHEME -workspace WORKSPACE build 或xcodebuild -scheme SCHEME -project PROJECT build（将其中的大写单词换成你项目的对应名称），然后观察是否有相同的失败发生，这应该能生成足够的失败信息来解决问题。
稳定版发布的标签

Carthage使用语义化标签来发布稳定版本。如1.2.0，如带有字母则是不受支持的版本（如1.2-alpha-1）.
CarthageKit

大多数carthage命令行工具的功能都封装在一个名为CarthageKit的框架中。

如果你希望将Carthage作为另一个工具的一部分，或者希望扩展Carthage的功能，可以看看CarthageKit的源码，检查API是否符合你的需求。
授权协议

Carthage使用MIT开源协议授权发布。


2. xcode快捷键


运行 cmd+r 
停止运行 cmd+. 
查找cmd+f 
全局查找shift+cmd+f 
打开文件shift+cmd+o 
编译 cmd+b
回到上一个文件 ctrl+cmd+left
回到下一个文件 ctrl+cmd+right
新建文件 cmd+n
新建工程 shift+cmd+n
清理工程 shift+cmd+k
跳转到头文件 ctrl+cmd+up
opt+left click

3. IBDesignable

IBDesignable
主要作用：可以显示出来你使用代码写的界面。 

IBInspectable
主要作用：使view内的变量可视化，并且可以修改后马上看到 

4. storyboard管理


5. 三方库(swift约束)

Masonry、SnapKit 

6. get body中能否添加数据

可以，理论上get和post没有任何区别，只是规范给他们分别对待了。

7. runtime

最近在找工作，Objective-C中的Runtime是经常被问到的一个问题，几乎是面试大公司必问的一个问题。当然还有一些其他问题也几乎必问，例 如：RunLoop，Block，内存管理等。其他的问题如果有机会我会在其他文章中介绍。本篇文章主要介绍RunTime。

RunTime简称运行时。就是系统在运行的时候的一些机制，其中最主要的是消息机制。对于C语言，函数的调用在编译的时候会决定调用哪个函数（ C语言的函数调用请看这里 ）。编译完成之后直接顺序执行，无任何二义性。OC的函数调用成为消息发送。属于动态调用过程。在编译的时候并不能决定真正调用哪个函数（事实证明，在编 译阶段，OC可以调用任何函数，即使这个函数并未实现，只要申明过就不会报错。而C语言在编译阶段就会报错）。只有在真正运行的时候才会根据函数的名称找 到对应的函数来调用。

那OC是怎么实现动态调用的呢？下面我们来看看OC通过发送消息来达到动态调用的秘密。假如在OC中写了这样的一个代码：

	[obj makeText];

其中obj是一个对象，makeText是一个函数名称。对于这样一个简单的调用。在编译时RunTime会将上述代码转化成
	
	objc_msgSend(obj,@selector(makeText));

首先我们来看看obj这个对象，iOS中的obj都继承于NSObject。

	@interface NSObject <nsobject> {
	    Class isa  OBJC_ISA_AVAILABILITY;
	}</nsobject>

在NSObjcet中存在一个Class的isa指针。然后我们看看Class：
	
	typedef struct objc_class *Class;
	struct objc_class {
	  Class isa; // 指向metaclass
	   
	  Class super_class ; // 指向其父类
	  const char *name ; // 类名
	  long version ; // 类的版本信息，初始化默认为0，可以通过runtime函数class_setVersion和class_getVersion进行修改、读取
	  long info; // 一些标识信息,如CLS_CLASS (0x1L) 表示该类为普通 class ，其中包含对象方法和成员变量;CLS_META (0x2L) 表示该类为 metaclass，其中包含类方法;
	  long instance_size ; // 该类的实例变量大小(包括从父类继承下来的实例变量);
	  struct objc_ivar_list *ivars; // 用于存储每个成员变量的地址
	  struct objc_method_list **methodLists ; // 与 info 的一些标志位有关,如CLS_CLASS (0x1L),则存储对象方法，如CLS_META (0x2L)，则存储类方法;
	  struct objc_cache *cache; // 指向最近使用的方法的指针，用于提升效率；
	  struct objc_protocol_list *protocols; // 存储该类遵守的协议
	    }


我们可以看到，对于一个Class类中，存在很多东西，下面我来一一解释一下：

Class isa：指向metaclass，也就是静态的Class。一般一个Obj对象中的isa会指向普通的Class，这个Class中存储普通成员变量和对 象方法（“-”开头的方法），普通Class中的isa指针指向静态Class，静态Class中存储static类型成员变量和类方法（“+”开头的方 法）。

Class super_class:指向父类，如果这个类是根类，则为NULL。

下面一张图片很好的描述了类和对象的继承关系：

注意：所有metaclass中isa指针都指向跟metaclass。而跟metaclass则指向自身。Root metaclass是通过继承Root class产生的。与root class结构体成员一致，也就是前面提到的结构。不同的是Root metaclass的isa指针指向自身。

Class类中其他的成员这里就先不做过多解释了，下面我们来看看：

@selector (makeText)：这是一个SEL方法选择器。SEL其主要作用是快速的通过方法名字（makeText）查找到对应方法的函数指针，然后调用其函 数。SEL其本身是一个Int类型的一个地址，地址中存放着方法的名字。对于一个类中。每一个方法对应着一个SEL。所以iOS类中不能存在2个名称相同 的方法，即使参数类型不同，因为SEL是根据方法名字生成的，相同的方法名称只能对应一个SEL。

下面我们就来看看具体消息发送之后是怎么来动态查找对应的方法的。

首先，编译器将代码[obj makeText];转化为objc_msgSend(obj, @selector (makeText));，在objc_msgSend函数中。首先通过obj的isa指针找到obj对应的class。在Class中先去cache中 通过SEL查找对应函数method（猜测cache中method列表是以SEL为key通过hash表来存储的，这样能提高函数查找速度），若 cache中未找到。再去methodList中查找，若methodlist中未找到，则取superClass中查找。若能找到，则将method加 入到cache中，以方便下次查找，并通过method中的函数指针跳转到对应的函数中去执行。

Runtime是想要做好iOS开发，或者说是真正的深刻的掌握OC这门语言所必需理解的东西。最近在学习Runtime，有自己的一些心得，整理如下，

一为 查阅方便

二为 或许能给他人一些启发，

三为 希望得到大家对这篇整理不足之处的一些指点。

什么是Runtime

我们写的代码在程序运行过程中都会被转化成runtime的C代码执行，例如[target doSomething];会被转化成objc_msgSend(target, @selector(doSomething));。

OC中一切都被设计成了对象，我们都知道一个类被初始化成一个实例，这个实例是一个对象。实际上一个类本质上也是一个对象，在runtime中用结构体表示。

相关的定义：
	
	/// 描述类中的一个方法
	typedef struct objc_method *Method;
	/// 实例变量
	typedef struct objc_ivar *Ivar;
	/// 类别Category
	typedef struct objc_category *Category;
	/// 类中声明的属性
	typedef struct objc_property *objc_property_t;

类在runtime中的表示
	
	//类在runtime中的表示
	struct objc_class {
	    Class isa;//指针，顾名思义，表示是一个什么，
	    //实例的isa指向类对象，类对象的isa指向元类
	#if !__OBJC2__
	    Class super_class;  //指向父类
	    const char *name;  //类名
	    long version;
	    long info;
	    long instance_size
	    struct objc_ivar_list *ivars //成员变量列表
	    struct objc_method_list **methodLists; //方法列表
	    struct objc_cache *cache;//缓存
	    //一种优化，调用过的方法存入缓存列表，下次调用先找缓存
	    struct objc_protocol_list *protocols //协议列表
	    #endif
	} OBJC2_UNAVAILABLE;
	/* Use `Class` instead of `struct objc_class *` */

获取列表

有时候会有这样的需求，我们需要知道当前类中每个属性的名字（比如字典转模型，字典的Key和模型对象的属性名字不匹配）。

我们可以通过runtime的一系列方法获取类的一些信息（包括属性列表，方法列表，成员变量列表，和遵循的协议列表）。
	
	  unsigned int count;
	    //获取属性列表
	    objc_property_t *propertyList = class_copyPropertyList([self class], &count);
	    for (unsigned int i=0; i<count; i++) {         const char *propertyname =" property_getName(propertyList[i]);"         nslog(@"property----="">%@", [NSString stringWithUTF8String:propertyName]);
	    }
	    //获取方法列表
	    Method *methodList = class_copyMethodList([self class], &count);
	    for (unsigned int i; i<count; i++) {         method method =" methodList[i];"         nslog(@"method----="">%@", NSStringFromSelector(method_getName(method)));
	    }
	    //获取成员变量列表
	    Ivar *ivarList = class_copyIvarList([self class], &count);
	    for (unsigned int i; i<count; i++) {         ivar myivar =" ivarList[i];"         const char *ivarname =" ivar_getName(myIvar);"         nslog(@"ivar----="">%@", [NSString stringWithUTF8String:ivarName]);
	    }
	    //获取协议列表
	    __unsafe_unretained Protocol **protocolList = class_copyProtocolList([self class], &count);
	    for (unsigned int i; i<count; i++) {         protocol *myprotocal =" protocolList[i];"         const char *protocolname =" protocol_getName(myProtocal);"         nslog(@"protocol----="">%@", [NSString stringWithUTF8String:protocolName]);
	    }</count; i++) {></count; i++) {></count; i++) {></count; i++) {>

在Xcode上跑一下看看输出吧，需要给你当前的类写几个属性，成员变量，方法和协议，不然获取的列表是没有东西的。

注意，调用这些获取列表的方法别忘记导入头文件#import。

方法调用

让我们看一下方法调用在运行时的过程（参照前文类在runtime中的表示）

如果用实例对象调用实例方法，会到实例的isa指针指向的对象（也就是类对象）操作。

如果调用的是类方法，就会到类对象的isa指针指向的对象（也就是元类对象）中操作。

    首先，在相应操作的对象中的缓存方法列表中找调用的方法，如果找到，转向相应实现并执行。

    如果没找到，在相应操作的对象中的方法列表中找调用的方法，如果找到，转向相应实现执行

    如果没找到，去父类指针所指向的对象中执行1，2.

    以此类推，如果一直到根类还没找到，转向拦截调用。

    如果没有重写拦截调用的方法，程序报错。

以上的过程给我带来的启发：

    重写父类的方法，并没有覆盖掉父类的方法，只是在当前类对象中找到了这个方法后就不会再去父类中找了。

    如果想调用已经重写过的方法的父类的实现，只需使用super这个编译器标识，它会在运行时跳过在当前的类对象中寻找方法的过程。

拦截调用

在方法调用中说到了，如果没有找到方法就会转向拦截调用。

那么什么是拦截调用呢。

拦截调用就是，在找不到调用的方法程序崩溃之前，你有机会通过重写NSObject的四个方法来处理。
	
	+ (BOOL)resolveClassMethod:(SEL)sel;
	+ (BOOL)resolveInstanceMethod:(SEL)sel;
	//后两个方法需要转发到其他的类处理
	- (id)forwardingTargetForSelector:(SEL)aSelector;
	- (void)forwardInvocation:(NSInvocation *)anInvocation;

    第一个方法是当你调用一个不存在的类方法的时候，会调用这个方法，默认返回NO，你可以加上自己的处理然后返回YES。

    第二个方法和第一个方法相似，只不过处理的是实例方法。

    第三个方法是将你调用的不存在的方法重定向到一个其他声明了这个方法的类，只需要你返回一个有这个方法的target。

    第四个方法是将你调用的不存在的方法打包成NSInvocation传给你。做完你自己的处理后，调用invokeWithTarget:方法让某个target触发这个方法。

动态添加方法

重写了拦截调用的方法并且返回了YES，我们要怎么处理呢？

有一个办法是根据传进来的SEL类型的selector动态添加一个方法。

首先从外部隐式调用一个不存在的方法：
	
	//隐式调用方法
	[target performSelector:@selector(resolveAdd:) withObject:@"test"];

然后，在target对象内部重写拦截调用的方法，动态添加方法。
	
	void runAddMethod(id self, SEL _cmd, NSString *string){
	    NSLog(@"add C IMP ", string);
	}
	+ (BOOL)resolveInstanceMethod:(SEL)sel{
	    //给本类动态添加一个方法
	    if ([NSStringFromSelector(sel) isEqualToString:@"resolveAdd:"]) {
	        class_addMethod(self, sel, (IMP)runAddMethod, "v@:*");
	    }
	    return YES;
	}

其中class_addMethod的四个参数分别是：

    Class cls 给哪个类添加方法，本例中是self

    SEL name 添加的方法，本例中是重写的拦截调用传进来的selector。

    IMP imp 方法的实现，C方法的方法实现可以直接获得。如果是OC方法，可以用+ (IMP)instanceMethodForSelector:(SEL)aSelector;获得方法的实现。

    "v@:*"方法的签名，代表有一个参数的方法。

关联对象

现在你准备用一个系统的类，但是系统的类并不能满足你的需求，你需要额外添加一个属性。

这种情况的一般解决办法就是继承。

但是，只增加一个属性，就去继承一个类，总是觉得太麻烦类。

这个时候，runtime的关联属性就发挥它的作用了。

	//首先定义一个全局变量，用它的地址作为关联对象的key
	static char associatedObjectKey;
	//设置关联对象
	objc_setAssociatedObject(target, &associatedObjectKey, @"添加的字符串属性", OBJC_ASSOCIATION_RETAIN_NONATOMIC); //获取关联对象
	NSString *string = objc_getAssociatedObject(target, &associatedObjectKey);
	NSLog(@"AssociatedObject = %@", string);
	
objc_setAssociatedObject的四个参数：

    id object给谁设置关联对象。

    const void *key关联对象唯一的key，获取时会用到。

    id value关联对象。

    objc_AssociationPolicy关联策略，有以下几种策略：
	
	enum {
	    OBJC_ASSOCIATION_ASSIGN = 0,
	    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, 
	    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,
	    OBJC_ASSOCIATION_RETAIN = 01401,
	    OBJC_ASSOCIATION_COPY = 01403 
	};

如果你熟悉OC，看名字应该知道这几种策略的意思了吧。

    objc_getAssociatedObject的两个参数。

    id object获取谁的关联对象。

const void *key根据这个唯一的key获取关联对象。

其实，你还可以把添加和获取关联对象的方法写在你需要用到这个功能的类的类别中，方便使用。
	
	//添加关联对象
	- (void)addAssociatedObject:(id)object{
	    objc_setAssociatedObject(self, @selector(getAssociatedObject), object, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
	}
	//获取关联对象
	- (id)getAssociatedObject{
	    return objc_getAssociatedObject(self, _cmd);
	}

注意：这里面我们把getAssociatedObject方法的地址作为唯一的key，_cmd代表当前调用方法的地址。

方法交换

方法交换，顾名思义，就是将两个方法的实现交换。例如，将A方法和B方法交换，调用A方法的时候，就会执行B方法中的代码，反之亦然。

话不多说，这是参考Mattt大神在NSHipster上的文章自己写的代码。
		
	#import "UIViewController+swizzling.h"
	#import @implementation UIViewController (swizzling)
	//load方法会在类第一次加载的时候被调用
	//调用的时间比较靠前，适合在这个方法里做方法交换
	+ (void)load{
	    //方法交换应该被保证，在程序中只会执行一次
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        //获得viewController的生命周期方法的selector
	        SEL systemSel = @selector(viewWillAppear:);
	        //自己实现的将要被交换的方法的selector
	        SEL swizzSel = @selector(swiz_viewWillAppear:);
	        //两个方法的Method
	        Method systemMethod = class_getInstanceMethod([self class], systemSel);
	        Method swizzMethod = class_getInstanceMethod([self class], swizzSel);
	        //首先动态添加方法，实现是被交换的方法，返回值表示添加成功还是失败
	        BOOL isAdd = class_addMethod(self, systemSel, method_getImplementation(swizzMethod), method_getTypeEncoding(swizzMethod));
	        if (isAdd) {
	            //如果成功，说明类中不存在这个方法的实现
	            //将被交换方法的实现替换到这个并不存在的实现
	            class_replaceMethod(self, swizzSel, method_getImplementation(systemMethod), method_getTypeEncoding(systemMethod));
	        }else{
	            //否则，交换两个方法的实现
	            method_exchangeImplementations(systemMethod, swizzMethod);
	        }
	    });
	}
	- (void)swiz_viewWillAppear:(BOOL)animated{
	    //这时候调用自己，看起来像是死循环
	    //但是其实自己的实现已经被替换了
	    [self swiz_viewWillAppear:animated];
	    NSLog(@"swizzle");
	}
	@end

在一个自己定义的viewController中重写viewWillAppear
	
	- (void)viewWillAppear:(BOOL)animated{
	    [super viewWillAppear:animated];
	    NSLog(@"viewWillAppear");
	}

Run起来看看输出吧！

我的理解：

    方法交换对于我来说更像是实现一种思想的最佳技术：AOP面向切面编程。

    既然是切面，就一定不要忘记，交换完再调回自己。

    一定要保证只交换一次，否则就会很乱。

    最后，据说这个技术很危险，谨慎使用。

完

前言:

    对于一个大项目而言，最烦恼的就是在众多界面难以找到对应的viewController,要改个东西都要花好长的时间去找对应的类。

    特别是当你接手一个大项目的时候，对整体的业务逻辑不熟悉，整体的架构体系不熟悉，让你修复某个页面的BUG，估计你找这个页面所对应的viewController都要找好久。

思考

    能否有一种方式可以快速让你上手一个大项目？快速找到某个页面所对应的viewController ?

思路

    在每一个页面出现的时候，都打印出哪个类即将出现，如下图所示

解决方案

    方案1

        整个项目中建立一个基类的viewController，然后将项目中所有的viewController都继承于基类的viewController，然后重写基类中的viewWillAppear方法

	- (void)viewWillAppear:(BOOL)animated {
	    [super viewWillAppear:animated];
	    NSString *className = NSStringFromClass([self class]);
	    NSLog(@"%@ will appear", className);
	}

    方案2

        给UIViewContoller建立一个分类，在分类里进行方法的交换，既保留了原本的方法，又有打印信息

	//
	//  UIViewController+Swizzling.m
	//  CollectionsOfExample
	//
	//  Created by mac on 16/10/1.
	//  Copyright ? 2016年 chenfanfang. All rights reserved.
	//
	
	#import "UIViewController+Swizzling.h"
	
	#import @implementation UIViewController (Swizzling)
	
	+ (void)load {
	
	    //我们只有在开发的时候才需要查看哪个viewController将出现
	    //所以在release模式下就没必要进行方法的交换
	#ifdef DEBUG
	
	    //原本的viewWillAppear方法
	    Method viewWillAppear = class_getInstanceMethod(self, @selector(viewWillAppear:));
	
	    //需要替换成 能够输出日志的viewWillAppear
	    Method logViewWillAppear = class_getInstanceMethod(self, @selector(logViewWillAppear:));
	
	    //两方法进行交换
	    method_exchangeImplementations(viewWillAppear, logViewWillAppear);
	
	#endif
	
	}
	
	- (void)logViewWillAppear:(BOOL)animated {
	
	    NSString *className = NSStringFromClass([self class]);
	
	    //在这里，你可以进行过滤操作，指定哪些viewController需要打印，哪些不需要打印
	    if ([className hasPrefix:@"UI"] == NO) {
	        NSLog(@"%@ will appear",className);
	    }
	
	
	    //下面方法的调用，其实是调用viewWillAppear
	    [self logViewWillAppear:animated];
	}
	
	@end

优缺点分析

    方案1  适用于一个新项目，从零开始搭建的项目，建立一个基类controller,这种编程思想非常可取。但对于一个已经成型的项目，则方案一行不通，你总不能建议一个基类，让后将所有的controller继承的类都改成基类吧？这工程量太大，太麻烦。

    方案2 不论是从零开始搭建的项目，还是已经成型的项目，方案2都适用。
[runtime 文脏](http://www.jianshu.com/p/364eab29f4f5)

8. UIButton add block(closure)

方法一：继承UIButton

UIButtonBlock.h文件 如下

	#import <UIKit/UIKit.h>
	typedef void (^ClickActionBlock) (id obj);
	
	@interface UIButtonBlock : UIButton
	@property (nonatomic,strong)ClickActionBlock caBlock;
	
	- (void)initWithBlock:(ClickActionBlock)clickBlock for:(UIControlEvents)event;
	
	@end
	
	UIButtonBlock.m文件如下：
	
	#import "UIButtonBlock.h"
	
	@implementation UIButtonBlock
	- (void)initWithBlock:(ClickActionBlock)clickActionBlock for:(UIControlEvents)event{
		[self addTarget:self action:@selector(goAction:) forControlEvents:event];
		self.caBlock = clickActionBlock;
	}
	- (void)goAction:(UIButton *)btn{
		self.caBlock(btn);
	}

附使用方法。。。。。我这里是用storyboard拖出来的按钮。首先要在storyboard里的Button关联这个UIButtonBlock这个类

然后就是使用：

……

 

    [self.clickButton initWithBlock:^(id obj) {

        NSLog(@"继承之UIButton============%@",obj);

    } for:UIControlEventTouchUpInside];

有同学觉得多此一举，这里不作解释。

方法二：给UIButton添加一个分类

UIButton+Block.h文件如下

 

	#import <UIKit/UIKit.h>
	
	typedef void (^ClickActionBlock) (id obj);
	
	@interface UIButton (Block)
	
	- (void)initWithBlock:(ClickActionBlock)clickBlock for:(UIControlEvents)event;
	@end
	
	UIButton+Block.m文件如下
	
	#import "UIButton+Block.h"
	
	#import <objc/runtime.h>
	static id key;
	@implementation UIButton (Block)
	- (void)initWithBlock:(ClickActionBlock)clickBlock for:(UIControlEvents)event{
		 objc_setAssociatedObject(self, &key, clickBlock, OBJC_ASSOCIATION_COPY_NONATOMIC);
		[self addTarget:self action:@selector(goAction:) forControlEvents:event];
	}
	- (void)goAction:(UIButton *)sender{
		ClickActionBlock block = (ClickActionBlock)objc_getAssociatedObject(self, &key);
		if (block) {
			 block(sender);
		}
	}

使用方法：

	[self.blockButton initWithBlock:^(id obj) {
		NSLog(@"Runtime block============%@",obj);
	} for:UIControlEventTouchUpInside ];

    

方法一：使用注意如果手写的UIButton需要在UIButtonBlock中再写一个初始化方法。如果是从xib拖出来的是需要关联的。

两种方法都实现了通过块来实现UIButton的addtarget方法中的@select方法的回调。代码比较粗糙大家将就着看着

9. git(add、commit、push)

git commit操作的是本地库，git push操作的是远程库。

git commit是将本地修改过的文件提交到本地库中。
git push是将本地库中的最新信息发送给远程库。

add是选择提交哪些文件，可以不选择不想提交的文件。

10. 支付宝sdk安全
11. http状态码 200 400 500
12. 应用里有哪几个文件夹，iTunes会保存的是哪个文件夹

Documents: 最常用的目录，iTunes同步该应用时会同步此文件夹中的内容，适合存储重要数据。
Library/Caches: iTunes不会同步此文件夹，适合存储体积大，不需要备份的非重要数据。
Library/Preferences: iTunes同步该应用时会同步此文件夹中的内容，通常保存应用的设置信息。

13. gcd nsoperationqueue thread(区别)各有什么优势
14. tableview的重用，会有什么问题会导致卡顿
15. 对OC的理解
16. 控制链传递
17. 动画