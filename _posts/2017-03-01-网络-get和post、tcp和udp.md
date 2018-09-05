---
title: 网络-get和post、tcp和udp
description: This is an introduction to difference between get and post
header: 网络-get和post、tcp和udp
---

网络中关于这些的介绍一搜一大篇，但是我并不是专门搞网络的，所以我觉得我只要记住我经常会用到的一些知识点。然后我总结了一下：

HTTP定义了服务器交互的不同方法，*基本*的有四种：put delete get post

get一般用于获取资源信息，post一般用于更新资源信息。

根据http规范，get用于获取信息，所以应该是安全和幂等的。

安全就是指该操作只用于获取信息而不会修改信息，也就是get不会产生副作用。

幂等是指对于同一UIRL获取到的信息应该是相同的。

而根据http规范，post是可能改变服务器上资源的请求。

但在实际上操作的时候，很多人为了图方便更新资源可能会有get，因为使用post需要提交form表单。

get提交是把数据放在URL里面的，而post提交是将数据放在HTTP包的包体中。

传输数据数据大小的限制：http协议是没有对数据大小进行限制的，http规范也没有对URL长度进行限制。

但是对于浏览器或者服务器来说，会对get进行长度限制，如果浏览器没有进行限制的话，一般操作系统也会进行限制的。

post由于不是URL传值(可以人肉手输)，所以一般是没有限制的，但是服务器可能会进行限制。不适合用于传输大型数据。

post传输数据不是直接显示在URL中所以相对get来说*稍微*安全一点。如果被抓包那就没有区别了。

get可以被缓存，post不可以被缓存。

GET和POST是由HTTP协议定义的。在HTTP协议中，Method和Data（URL， Body， Header）是正交的两个概念，也就是说，使用哪个Method与应用层的数据如何传输是没有相互关系的。HTTP没有要求，如果Method是POST数据就要放在BODY中。也没有要求，如果Method是GET，数据（参数）就一定要放在URL中而不能放在BODY中。

那么，网上流传甚广的这个说法是从何而来的呢？我在HTML标准中，找到了相似的描述。这和网上流传的说法一致。但是这只是HTML标准对HTTP协议的用法的约定。怎么能当成GET和POST的区别呢？

而且，现代的Web Server都是支持GET中包含BODY这样的请求。虽然这种请求不可能从浏览器发出，但是现在的Web Server又不是只给浏览器用，已经完全地超出了HTML服务器的范畴了。

###2018.09.05 新加的理解
###GET和POST实质

GET和POST是什么？HTTP协议中的两种发送请求的方法。

HTTP是什么？HTTP是基于TCP/IP的关于数据如何在万维网中如何通信的协议。

HTTP的底层是TCP/IP。所以GET和POST的底层也是TCP/IP，也就是说，GET和POST都是TCP链接。GET和POST能做的事情是一样一样的。你要给GET加上request body，给POST带上url参数，技术上完全是通行的。
>在我大万维网世界中，TCP就像骑车，我们用TCP来运输数据，它很可靠，从来不会法神丢件少件的现象。但是如果路上跑的全是看起来一模一样的的汽车，那这个世界看起来是一团混乱，送急件的汽车可能被前面满载货物的汽车拦堵在路上，整个交通系统一定会瘫痪，为了避免这种情况发生，交通规则HTTP诞生了。HTTP给汽车运输定了好几个服务类别，有GET、POST、PUT、DELETE等。
>HTTP规定，当执行GET请求的时候，要给汽车贴上GET标签(设置method为GET)，而且要求把传送的数据放在车顶上(url中)以方便记录。如果是POST请求，就要在车上贴上POST的标签，并把货物放在车厢里。当然，你也可以在GET的时候往车厢内偷偷藏点货物，但是这很不光彩。也可以在POST的时候在车顶上也放一些数据，让人觉得傻乎乎的，HTTP只是个行为准则，而TCP才是GET和POST怎么实现的基本。

但是我们只看到HTTP对GEP和POST参数的传送渠道(url还是request body)提出了要求。关于参数大小的限制又是从哪来的呢？
>在我大万维网世界中，还有另一个重要角色：运输公司。不同的浏览器(发起http请求)和服务器(接收http请求)就是不同的运输公司。虽然理论上，你可以在车顶上无限的堆货物(url中无限加参数)。但是运输公司可不傻，装货和卸货也是有很大成本的，他们会限制单次运输量来控制风险，数据量太大对浏览器和服务器都是很大负担。业界不成文的规定是，(大多数)浏览器通常都会限制url长度在2k个字节，而(大多数)服务器最多处理64k大小的url。超过的部分，恕不处理。如果你用GET服务，在request body偷偷藏了数据，不同服务器的处理方式也是不同的，有些服务器会帮你卸货，读出数据，有些服务器直接忽略，所以，虽然GET可以带request body，也不能保证一定能被接收到哦。好了，现在你知道，GET和POST本质上就是TCP链接，并无差别。但是由于HTTP的规定，和浏览器/服务器的限制，导致他们在应用过程中体现出一些不同。

###GET和POST还以一个重大区别
GET产生一个TCP数据包，POST产生两个数据包
* 对于GET方式的请求，浏览器会把http header和data一起发送出去，服务器响应200(返回数据)；
* 而对于POST，浏览器先发送header，服务器响应100continue，浏览器再发送data，服务器响应200 ok(返回数据)。

###最后不要混用

因为post需要两步，时间上消耗的要多一点，看起来GET和POST更有效。因此Yahoo团队有推荐用GET替换POST来优化网站性能，但这是一个坑！跳入需谨慎。为什么？

1. GET和POST都有自己的语义，不能随便混用。
2. 据研究，在网络环境好的情况下，发一次包的时间和发两次包的时间差基本可以忽略，而在网络差的情况下，两次包的TCP在验证数据包完整性上，有非常大的有点。
3. 并不是所有浏览器都会在POST中发送两次，__firefox就只发送一次。__

###新的发现:

上面的GET和POST的重大区别是错误的，也就是说__GET产生一个数据包POST产生两个数据包的说法是错误的__。

RFC里讲的很明白: 100 continue只有在请求里带了

	Expect: 100-continue

header的时候才有意义。
> When the request contains an Expect header field that inckudes a 100-continue exoectation, the 100 response indicates that the server wishes to receive the request payload body, as described in Section 5.1.1. The client ought to continue sending the request adn discard the 100 response.
>  If the request did not contain an Expect header field containing the 100-continue exception, the client can simply discard this interim response.

而实际上，不论哪一种刘篮球，在发送POST的时候都没有带Expect头，server也自然不会发100continue。通过抓包发现，尽管会分两次，body就是紧随在header后面发送的，根本不存在__等待服务器相应__这一说。

[原文链接](https://zhuanlan.zhihu.com/p/25028045)


TCP和UDP:

tcp是面向连接的，经历三次握手，连接成功，断开连接要进行四次

udp是非连接的，因此可以同时向多个客户机同时发送相同的数据

tcp发送的数据是稳定的，能够保证数据的正确性，有序的

udp是无序的，可能会丢包，但是速度没有限制，限制的是带宽和计算机的能力和程序生成数据的速度。

tcp包头最小长度20字节，udp只有8个字节，所以开销很小。

小结TCP与UDP的区别：

1.基于连接与无连接；

2.对系统资源的要求（TCP较多，UDP少）；

3.UDP程序结构较简单；

4.流模式与数据报模式 ；

5.TCP保证数据正确性，UDP可能丢包，TCP保证数据顺序，UDP不保证。
>对于可靠性而言，没有绝对可靠，TCP可靠仅仅是在传输层实现了可靠，我也可以让UDP可靠啊，那么就要向上封装，在应用层实现可靠性。因此很多公司都不是直接用TCP和UDP，都是经过封装，满足业务需求。说到这里，就要提一下心跳包，在linux下有keep-alive系统自带的，但是默认时间很长，如果想使用的话可以setsockopt设置，我也可以在应用层实现一个简单心跳包，上次自己多开了一个线程来处理，还是爆头解决。

PS:网络分为7层，分别是物理层、数据链路层、网络层、传输层、会话层、表示层和应用层。http属于应用层。
tcp/ip协议是一个协议簇(只有四层)，里面包含着一大推协议

那么什么时候用TCP什么时候用UDP呢？

不管用TCP还是UDP，应用只看需求，对于TCP而言，注重的是可靠性，而不是实时性，如果我发送的数据很重要一点也不能出错，有延迟无所谓的话，那就TCP啊。UDP更加注重速度快，也就是实时性，对于可靠性要求并不那么高。


[别人的网络介绍总结1](http://www.cnblogs.com/jking10/p/5525519.html) [别人的网络介绍总结2](http://www.cnblogs.com/-yan/p/4962619.html)