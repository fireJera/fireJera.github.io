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

2. xcode快捷键
3. IBDesignable
4. storyboard管理
5. 三方库(swift约束)
6. get body中能否添加数据
7. runtime
8. UIButton add block(closure)
9. git(add、commit、push)
10. 支付宝sdk安全
11. http状态码 200 400 500
