假设一个服务器只有一个线程处理客户请求，四个客户端请求顺序发起：

- 第一个客户请求向服务器获取股票报价，但股票报价信息不在当前的服务器上存储，需要向其它服务器请求获取后再返回给客户端。

- 第二个客户请求向服务器获取CSS样式表，这个样式表在当前服务器的存储器中可直接获取，可以立即返回给客户端。

- 第三个客户请求向服务器获取用户信息，需要查询当前服务器上的用户信息数据库，查询执行完成后返回给客户端。

- 第四个客户请求向服务器获取HTML内容，这些HTML内容在当前服务器的存储器中可直接获取并返回给客户端。

如果用一个单线程服务器处理这四个请求，当第一个请求发起后，第二个请求需要在第一个请求处理完成后才能被处理到，因此四个请求是顺序执行，前一个处理完成才能处理后一个。这种顺序处理请求的方式容易发生阻塞，如果其中一个请求耗时太久，就会影响到后序几个请求。

如果用一个多线程服务器处理这四个请求，由于服务器同时可开启的线程数量是有上限的，线程间切换执行环境也会耗费资源，并且数据在多线程间访问时处理不当很容易发生错误，保证线程安全访问对于开发者很有挑战，很容易出错。多线程服务器一般会使用线程池的方式，虽然可以解决问题，但是效率不是最高的。

# 异步非阻塞IO

Vapor 建立在 SwiftNIO 之上，SwiftNIO 提供了异步非阻塞IO的处理方式，这也构成了 Vapor 的一个重要特性，刚开始学习 Vapor 的异步处理机制时会有些令人困惑。

!!! question "如何理解Vapor中的异步处理机制"
    如果在同一个单线程的服务器上用异步非阻塞IO的方式处理这四个请求，当第一个请求发起后，由于请求结果不能马上获取到，此时线程就会把第一个请求先放在一边，直接去处理后面的其它请求。SwiftNIO依赖操作系统内核通知事件，当网络请求数据返回时，系统内核会通知SwiftNIO，SwiftNIO再进行事件分发。被放在一边的第一个请求收到通知之后被恢复执行，把请求结果返回。从第一个请求被放在一边到被恢复执行的这段时间内，并不会阻塞服务器线程去处理后面其它的请求，这样就提高了执行效率。

假设有一个函数是这样的：

```swift
func getAllUsers() -> [User] {
    
    var users:[User]?
    
    // do some database queries async

    return uesrs
}
```
假设函数中的数据库查询操作是异步执行的，这个函数返回时，数据库的查询操作还没有完成，所以调用方在函数返回时并不能正常工作的。

这种情况下，只知道函数会返回一个数组[User]，却不清楚返回的具体时刻，所以需要改造一下返回类型，用Future<Type>这个范型结构承诺在将来的某个时刻返回对应类型的数据。

```swift 
func getAllUsers() -> Future<[User]> {
    // do some database queries
}
```

使用Future可能一开始会有点困惑，因为这个概念还不是很熟悉，不过使用一段时间就会适应，毕竟在Vapor中会有大量场景使用它。

当我们从一个函数获得一个Future返回时，实际上是想在这个Future有实际结果时执行一些操作，但这个Future在被获取时还没有产生实际值，所以我们需要给Future提供它在产生实际值时需要进行的操作对应的回调函数，让Future自己在产生实际值时调用相应的处理回调。

!!! warning "TODO： 这里需要一个更加清晰的解释Future机制"

--- 

和Future搭配使用的操作有以下几种：

- flatMap：CollectionType<ElemType> -> AnotherElemType
- map: CollectionType<ElemType> -> CollectionType<AnotherElemType>
- transform: 与map类似，不处理具体元素，直接变换为指定值
- flatten: 等所有Future都返回时执行
- do/catch: 用来捕获错误，但不是恢复错误
- catchMap/catchFlatMap: 捕获并修复错误
- always: 不管结果如何总会执行
- wait: 不能在主线程上使用
- request.future(_:)可以创建在同一个请求线程上使用的Future

!!! note "关于FlatMap和Map的理解"

    通过下面的代码，演示两种操作的不同之处：

    ```swift
    let number = [1, 2, 3, 4]

    let mapped = number.map { Array(repeating: $0, count: $0) }
    // [[1], [2, 2], [3, 3, 3], [4, 4, 4, 4]]

    let flatMapped = number.flatMap { Array(repeating: $0, count: $0) }
    // [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
    ```

    实际上`s.flatMap(transform)`与`Array(s.map(transform).joined())`是等价的。


在Vapor中一个Request就是一个Worker，相当于一个线程。

全局操作支持最多五个Future结果返回后执行。

对Future可以链式操作，用来避免过度嵌套。

!!! note "[SwiftNIO](https://github.com/apple/swift-nio)"
    是苹果的一个开源跨平台异步网络库，它用来管理连接和处理数据传输，管理着事件循环(EventLoop)，每一个事件循环对应一个线程。

    如果一个线程写入一个变量的同时有另外一个线程同时的对这个变量进行读或者写操作，那么就会产生竞争关系，有可能会使你的应用发生崩溃。传统的处理方式是给多个线程同时访问的变量各自加一把锁来使对变量的访问变的有序，从而消除竞争关系。线程在访问一个变量前先给这个变量上锁，表示此刻该变量正在使用，其它线程不能访问，等访问完成后对该变量解锁，以示其它线程可以继续对其进行访问。这是一种解决办法，但是存在缺点，就是使用起来很复杂，同时也会影响到程序的执行效率。

    另一种思路是把对同一个变量的访问都放在同一个线程里来进行。但这就要求这个处理读写操作的线程要能够把读写的结果返回给发起读写请求的线程中去。其实每一个事件循环(EventLoop)都可以看作是一个线程。 如果变量的读写操作所在的线程把读写结果返回给没有发起读写请求的其它线程，那么SwiftNIO就会用崩溃来避免程序发生不确定问题。要把读写操作的结果返回给发起读写请求的相关线程中去的功能就需要用到`Future`和`Promise`这两个概念了。

    `Future`是用来描述目前还不存在，但未来会存在的信息的一种数据结构。写异步代码时用`Future`来表示一个处理结果是成功还是失败，这两种结果确定会发生在未来，但是目前不知道会是哪一种。但是如果不知道未来结果到底是什么样子(可能成功，可能失败，也可能是其它的)，那就需要创建`Promise`了。

    `Promise`和`Future`都必须在事件循环(EventLoop)中创建，`Future`会被返回给产生它的事件循环中，一次只能代表一个结果，要么成功，要么失败，都算是处理完成状态。然后`Promise`创建时一定是完成状态的。