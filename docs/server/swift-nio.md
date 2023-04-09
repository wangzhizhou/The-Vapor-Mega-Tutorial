# SwiftNIO 概述

!!! info 
    Vapor是基于 [SwiftNIO][swift-nio-repo] 构建的，所以最好还是对它进行一些了解，不过这也不是必需的，如果感兴趣，可以了解一下。不感兴趣也可以直接跳过本页内容。

SwiftNIO 是一个跨平台异步事件驱动的网络应用框架，基于它可以快速开发高性能可维护的协议服务器和客户端。SwiftNIO是Swift生态中等价于Java生态下 [Netty] 的存在。

## 简介

SwiftNIO 是使用Swift语言构建高性能网络应用的基础层工具。主要是针对那些使用`每个网络链接对应一个线程`的并发模型，因为这种并发模型在网络链接并发量很大时会变的相对低效。SwiftNIO 为了解决这种低效问题，大量使用了非阻塞型IO模式，与常见的阻塞型IO模式相比，应用可以不必等待网络把数据发送出去或者接收回来后再进行操作，而是让操作系统内核在IO操作可以执行时通知SwiftNIO，来实现异步操作处理。

SwiftNIO的目标不是提供类似于Web框架的高级应用层解决方案，而是专注于为这些高级应用层提供底层网络能力。当要构建一个Web应用时，用户一般不会直接使用SwiftNIO，他们会使用一些Swift生态中提供的Web框架完成任务，这些Web框架可能下层依赖的网络层基础能力就是SwiftNIO提供的。

## SwiftNIO 的基础架构

SwiftNIO 通过 NIOCore 模块中定义的8种类型提供基础能力：

- EventLoopGroup， 协议
- EventLoop， 协议 
- Channel，协议
- ChannelHandler，协议
- Bootstrap，一组相关结构体
- ByteBuffer，结构体
- EventLoopFuture, 范型类
- EventLoopPromise，范型结构体

所有的SwiftNIO应用最终都是通过上面的8种类型构建出来的

### EventLoops & EventLoopGroups

EventLoop是SwiftNIO中的基本元素，它是用来等待IO事件的对象，当对应IO事件完成时，它会进行一些回调操作。通常使用SwiftNIO的应用会使用相对少量的EventLoop，一般单个CPU核心对应一到两个EventLoop, EventLoop通常在应用的整个生命周期中一直保持运行，不断的循环执行并分发事件。

多个EventLoop可以被集合成EventLoopGroup，EventLoopGroup可以为它管理的多个EventLoop分配任务，平衡各EventLoop的工作量，避免其中一些EventLoop过载，而另外一些EventLoop空闲。

目前SwiftNIO提供了EventLoopGroup协议的一种实现，以及EventLoop协议的两种实现。

- `MultiThreadedEventLoopGroup`可以使用`pthread`库创建多个线程，并为每一个创建的线程开启一个`SelectedEventLoop`运行。

- `SelectedEventLoop`是EventLoop协议的一种实现，它根据所在的系统平台不同，使用`kqueue`或`epoll`方式管理来自文件描述符的IO事件或者派发工作

- `EmbeddedEventLoop`是EventLoop协议的另一种实现，是EventLoop的一个仿制品，主要用来写测试用例

### Channels & Channel Handlers & Channel Pipelines & Channel Contexts

SwiftNIO用户使用EventLoop的主要场景是创建EventLoopPromise类型或者调度任务，但他们需要花大量精力来处理`Channel`和`ChannelHandler`

在SwiftNIO应用中，每一个被处理的文件描述符都需要与一个Channel结合，Channel拥有这个文件描述符，并负责管理它的生命周期，同时也需要处理这个文件描述符相关的事件。当操作系统内核通知EventLoop一个文件描述符相关的事件时，EventLoop就会通知拥有这个文件描述符的Channel对象

Channel本身并没有太大作用，它主要用来传递数据。ChannelPipeline是由一系列ChannelHander对象组成，ChannelPipeline用来处理Channel上传递过来的数据，相当于数据处理流水线。

ChannelHandler可以通过ChannelHandlerContext了解自己在ChannelPipeline中的执行环境，获取自己前后结点对应的ChannelHander对象，保证事件可以在ChannelPipeline中双向流动。SwiftNIO中内置了一些ChannelHandler，用来进行特定目的的数据处理。

SwiftNIO也内置了一些Channel协议的实现，例如：ServerSocketChannel可以用来接受远端的Socket连接，SocketChannel用来处理TCP连接，DatagramChannel用来处理UDP数据包。EmbeddedChannel用来写测试用例

ChannelPipeline是线程安全的，ChannelPipeline中的所有代码逻辑都被放到同一个线程中执行，所以为了不阻塞ChannelPipeline的执行，需要每一个ChannelHandler中的逻辑都不能执行阻塞型的代码逻辑，否则会影响整个ChannelPipeline的执行效率

### Bootstrap

使用EventLoop也可以直接注册和配置Channel, SwiftNIO为了简化Channel的创建，也提供了一些内置对象，例如ServerBootstrap用来配置监听Channel，ClientBootstrap用来注册客户端TCP通道，DatagramBootstrap用来注册UDP通道

### ByteBuffer

SwiftNIO工作过程中，涉及到很多数据缓存区的处理操作，为了处理方便，提供了一些高性能数据结构。ByteBuffer就是一种快速的支持写时复制特性的数据缓存区数据结构。ByteBuffer提供了很多有用的特性，可以在非安全模式下使用，也可以关闭边界检查来提高执行性能。大部分场景，还是优先推荐在安全模式下使用ByteBuffer数据结构

### Promises & Futures

并发代码和同步代码主要的区别就是执行结果不能立刻返回。SwiftNIO提供了EventLoopPromise和EventLoopFuture来处理需要异步返回的操作。

EventLoopFuture本质上是函数返回值的占位容器。每一个EventLoopFuture都对应一个EventLoopPromise。当函数的返回值确定下来时，会通过EventLoopPromise把函数实际的返回值塞入EventLoopFuture对象中。

如果通过轮询的方式检查EventLoopFuture是否完成，会非常低效。因此EventLoopFuture内部维护了一个回调列表。用户可以把自己的回调塞入EventLoopFuture内部维护的回调列表中，当EventLoopFuture获取一个返回结果时，会执行回调列表中的回调代码逻辑，这样用户就能获取到函数执行的结果了。同时为了线程安全，EventLoopFuture的回调列表会被保证在与其对应的EventLoopPromise相同的EventLoop上执行，这样就能保证数据的同步访问，避免多线程问题。

!!! Tip
    如果看完这篇介绍，还是有点懵懂的话，下一步，就可以拉下[swift-nio][swift-nio-repo]的仓库源码，看看EventLoopPromise和EventLoopFuture的具体实现了。写代码就像写文章一样，能够读懂优秀仓库的代码，就相当于可以读懂名著一样，对个人提升来说是最快和最直接的。

[swift-nio-repo]: <https://github.com/apple/swift-nio>
[swift-nio-overview]: <https://github.com/apple/swift-nio#conceptual-overview>
[Netty]: <https://netty.io/>