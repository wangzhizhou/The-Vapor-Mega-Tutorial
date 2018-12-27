Vapor定义了`KeyedCache`协议，用来描述各各路缓存机制需要满足的功能，本身定义很简单

```swift
public protocol KeyedCache {

    func get<D>(_ key: String, as decodable: D.Type) -> Future<D?> where D: Decodable

    func set<E>(_ key: String, to encodable: E) -> Future<Void> where E: Encodable

    func remove(_ key: String) -> Future<Void>
}
```

# 内存缓存

存放数据在程序运行时分配的内存空间中，由于没有外部依赖，所以这些缓存数据非常适合用来开发和测试阶段中使用，不足之外是，如果程序结束，则这些在内存中的缓存数据就丢失了，并且不能在多个应用实例间共用。

`MemoryKeyedCache`可以在应用的所以事件循环中公用，这就是说，一旦有数据被缓存，那么所有的事件循环都能访问这个数据，非常有例于开发和测试阶段，但这样的缓存机制不是线程安全的，要求同步访问，因此在生产环境下，异步的访问这些内存缓存数据可能存在问题。


`DictionaryKeyedCache`只对事件循环自己有效，不同的事件循环有自己的DictionaryCache, 所以在一次会话中的不同请求间是不能公用这样一种缓存的。它适用于生产环境下的缓存。

# 数据库缓存

所有的数据库都可以用来作缓存，适用于那种应用重启后或多个应用实例之间公用缓存数据的情况。
可以使用主数据库来缓存数据，也可以开辟一个专门的数据库来作缓存，常用的用`Redis`数据库缓存。

`Redis`是一种开源缓存服务，它能作为web应用的缓存数据库，并被多数云服务商支持。它易于配置、快速、特性丰富。


当构建一个web应用时，向其它API请求数据时可能造成延时，如果向其它API请求数据耗时很多，那么自己写的这个API的响应也会变的很慢。除此之外，其它的API可能也作了流量访问限制，一段时间内不能超过最大访问次数，有了缓存的放，就可以减少访问这些其它API的次数，这会让你的接口变的响应更快速一些。

在计算机科学中缓存是很重要的概念，这里有几种方法来存储web应用的缓存数据：内存、Fluent数据库、Redis和其它。不同的缓存方式有不同的缓存失效策略：LRU(Least Recently Used)、RR(Random Replacement)、LIFO(Last In First Out)。
