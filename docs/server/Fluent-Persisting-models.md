`Fluent`是Vapor中和数据库交互的工具，即是ORM(Object Relational Mapping)对象关系映射。是在Vapor和数据库之间的一个抽象层。它的存在让我们操作数据库变得更加容易。我们不需要直接对数据库进行操作。

Models是Swfit中表示数据的术语，并且这个概念在Fluent中也是一致的。Models就是要存入数据库中的对象。

所有的Fluent数据模型都要遵守Codable协议，把类加上final可以获得性能提升，id会在存入数据库时填充。

自定义Model在数据库中ID名称和类型需要数据模型遵循Model协议。一般Fluent针对每种数据库已经定义了各种可以遵循的Model协议，如SQLiteModel和SQLiteUUIDModel以及SQLiteStringModel

要想把数据存入数据库，必须在数据库中建立相应的表，这种建表操作，Fluent通过migration来进行。Migration只会执行一次，不会创建已经存在的表。

`Vapor Cloud`可以托管你的应用，但是目前还不成熟，你可以试用，但一定要在用完后把布署置0，要不然会收托管费。停止托管时要重新布暑一次，命令如下：
``` bash
vapor cloud deploy --replicas=0
```