`Fluent`是Vapor中和数据库交互的工具，即是ORM(Object Relational Mapping)对象关系映射。是在Vapor和数据库之间的一个抽象层。它的存在让我们操作数据库变得更加容易。我们不需要直接对数据库进行操作。

Models是Swfit中表示数据的术语，并且这个概念在Fluent中也是一致的。Models就是要存入数据库中的对象。

所有的Fluent数据模型都要遵守Codable协议，把类加上final可以获得性能提升，id会在存入数据库时填充。Vapor提供了协议`Content`，它是对`Codable`的封装，可以方便在多种数据格式之间进行转换。

自定义Model在数据库中ID名称和类型需要数据模型遵循Model协议。一般Fluent针对每种数据库已经定义了各种可以遵循的Model协议，如SQLiteModel和SQLiteUUIDModel以及SQLiteStringModel

要想把数据存入数据库，必须在数据库中建立相应的表，这种建表操作，Fluent通过migration来进行。Migration只会执行一次，不会创建已经存在的表。

我们之后要创建一个缩略语相关的应用，这里可以先把数据模型建立起来：

```
$ touch Sources/App/Models/Acronym.swift
```

*Acronym.swift*

```swift
import Vapor
import FluentSQLite

final class Acronym: Codable { 
    var id: Int？
    var short: String 
    var long: String

    init(short: String, long: String) {
        self.short = short
        self.long = long
    }
}

extension Acronym: Model {
    typealias Database = SQLiteDatabase

    typealias ID = Int

    public static var idKey: IDKey = \Acronym.id
}

extension Acronym: SQLiteModel {}
extension Acronym: Migration {}
extension Acronym: Content {}
```

首先定义遵循`Codabel`协议的数据类型`Acronym`，它有两个字符串类型的属性，另一个属性`id`是留给数据库使用的，数据库在存储数据时，会赋值给每一个数据实例一个唯一标识。

然后我们又让数据模型遵循`Model`协议，其目的是为了告诉`Fluent`我们要使用哪种数据库来存储数据、数据实例唯一标识符的类型以及在数据模型中哪一个属性用来表示这个数据实例的唯一标识。每一种数据库其实已经提供了对应的协议类型，以简化操作，我们可以把这部分直接替换成:

```swift
extension Acronym: SQLiteModel {}
```

要把数据存入数据库，那么必须让数据库对应的建立一张存放这类数据的数据表，这个建表的任务我们可以让`Migration`来处理。但这要求我们的数据模型遵循协议`Migration`。对于简单的数据模型，我们只需要遵循这个协议就可以了，复杂的时时候可能需要我们自己实现一个`Mirgration`。
要让应用在启动时创建数据表，我们需要在`configure.swift`文件中进行配置：

```swift
migrations.add(model: Acronym.self, database: .sqlite)
```
一种类型的数据表只会创建一次，如果之后数据模型内部发生了调整也不会再重新创建新表了。