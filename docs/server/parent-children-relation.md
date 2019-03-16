父子关系描述一对一或者一对多的关系，例如： 一个人可以拥有一个宠物，一个人也可以拥有多个宠物，一个宠物只能有一个主人。这样的关系就是父子关系。即一个父亲可以用多个子女，但一个子女只能有一个父亲。

# 定义父子关系

因为孩子一定会有一个父亲，所以给孩子的数据模型中添加一个指向父亲的引用，就可以建立父子关系。

从子的一边定义就可以了, 添加了`userID: User.ID`属性，修改初始化函数。更新`updateHandler`

*Acronym.swift*
```swift
final class Acronym: Codable {
    var id: Int?
    var short: String
    var long: String
    
    // 一个用户创建一个缩略语
    var userID: User.ID
    
    init(short: String, long: String, userID: User.ID) {
        self.short = short
        self.long = long
        self.userID = userID
    }
}
```

*AcronymsController.swift*
```swift
func updateHandler(_ req: Request) throws -> Future<Acronym> {
    return try flatMap(to: Acronym.self, req.parameters.next(Acronym.self) req.content.decode(Acronym.self)) { (acronym, updateAcronym) - Future<Acronym> in
        acronym.short = updateAcronym.short
        acronym.long = updateAcronym.long
        acronym.userID = updateAcronym.userID
        
        return acronym.save(on: req)
    }
}
```

!!! note "关于数据库更新"
    因为`Migration`只会创建一次数据表，如果数据表已经创建，那么之后对数据模型的改动不会引起数据表的更新，所以需要重置数据库，重新创建数据表。还有一种方式是写数据库迁移规则，那个会涉及到数据库迁移，目前就是强势删库重建，因为目前的数据没有什么意义，这么做最方便。

# 从孩子侧获取父数据

由于我们在缩略词的数据结构中添加了它所属用户的id，所以可以通过`.parent`方法来查询数据库中的用户数据，从而获取用户信息。需要添加一部分代码用来获取父亲的信息

*Acronym.swift*
```swift
...
// 获取父关系数据
extension Acronym {
    var user: Parent<Acronym, User> { // User是Acronym的父亲
        return parent(\.userID)
    }
}
```

*AcronymsController.swift*
```swift
routeGroup.get(Acronym.parameter, "user", use: getUserHandler)
...
func getUserHandler(_ req: Request) throws -> Future<User> {
    return try req.parameters.next(Acronym.self).flatMap(to: User.self) { (acronym) -> Future<User> in
        return acronym.user.get(on: req)
    }
 }
```

# 获取子数据

从父数据中获取子数据，在这里就是获取一个用户创建的所有缩略词

*User.swift*
```swift 
//获取子数据信息
extension User {
    var acronyms: Children<User, Acronym> { // Acronym是User的孩子
        return children(\.userID)
    }
}
```

*UsersController.swift*
```swift
usersGroup.get(User.parameter, "acronyms", use: getAcronymsHandler)
...
func getAcronymsHandler(_ req: Request) throws -> Future<[Acronym]> {
    
    return try req.parameters.next(User.self).flatMap(to: [Acronym].self)  (user) -> Future<[Acronym]> in
        return try user.acronyms.query(on: req).all()
    }
}
```

# 外键约束

!!! note "SQLite 不支持外键约束"
    添加外键约束使用PostgreSQL验证
    
外键约束描述两张表之间的链接关系，在数据有效性验证中经常使用。使用外键约束有下面的好处：

- 确保不会创建出userID为无效用户的缩略语
- 在删除用户创建的所有缩略语之前不能删除该用户
- 在删除缩略语表之前无法删除用户表

外键约束需要在`Migration`中定义, 把原来Acronym.swift中对Migration的扩展改写如下：

```swift
// 在数据库中创建表及外键约束
extension Acronym: Migration {
    static func prepare(on connection: SQLiteConnection) -> Future<Void> {
        
        // 创建Acronym在数据库中的表
        return Database.create(self, on: connection) { (builder) in
            
            // 自动添加Acronym所有属性到数据表中
            try addProperties(to: builder)
            
            // 这一句添加了Acronym.userID到User.id的外键约束
            builder.reference(from: \.userID, to: \User.id)
        }
    }
}
```
同时因为添加了Acronym对User的外键约束，所以User对应的表应该先于Acronym表创建，所以需要调整一个confiure.swift中的表创建顺序： 
```swift
...
var migrations = MigrationConfig()
migrations.add(model: User.self, database: .sqlite)
migrations.add(model: Acronym.self, database: .sqlite)
services.register(migrations)
```

!!! note "删库重建"
    因为数据库中表的关系有变换，并且表只能创建一次，所以如果不删库重建，外键约束关系是不会生效的。目前我们还没有学习数据库迁移，并且我们的数据量不大也不重要。所以使用删库重建来使变改生效。每次表结构变化或关系变化，都执行此操作。