建立目录`Sources/App/Migrations`, 用来存放所有的数据库迁移相关的文件

# 给用户添加一个twitter帐号显示

*User.swift*

```swift
...
final class User: Codable {
    ...
    var twitterURL: String?
    
    init(name: String, username: String, password: String, twitterURL: String? = nil) {
        ...
        self.twitterURL = twitterURL
    }
    
    final class Public: Codable {
        ...
        var twitterURL: String?
        init(id: UUID?, name: String, username: String, twitterURL: String? = nil) {
            ...
            self.twitterURL = twitterURL
        }
    }
}

...

extension User {
    func convertToPublic() -> User.Public {
        return User.Public(id: self.id, name: self.name, username: self.username, twitterURL: self.twitterURL)
    }
}

...

extension User: Migration {
    static func prepare(on conn: PostgreSQLConnection) -> Future<Void> {
        return Database.create(self, on: conn, closure: { (builder) in
            builder.field(for: \.id, isIdentifier: true)
            builder.field(for: \.name)
            builder.field(for: \.username)
            builder.field(for: \.password)
            builder.unique(on: \.username)
        })
    }
}
```

*18-12-26-AddTwitterToUser.swift*

```swift
import FluentPostgreSQL
import Vapor

struct AddTwitterURLToUser: Migration {
    
    typealias Database = PostgreSQLDatabase

    static func prepare(on conn: PostgreSQLConnection) -> Future<Void> {
        return Database.update(User.self, on: conn) { builder in
            builder.field(for: \.twitterURL)
        }
    }
    
    static func revert(on conn: PostgreSQLConnection) -> Future<Void> {
        return Database.update(User.self, on: conn) { builder in
            builder.deleteField(for: \.twitterURL)
        }
    }
}
```

*configure.swift*

```swift
public func configure(_ config: inout Config, _ env: inout Environment, _ services: inout Services) throws {
    ...
    migrations.add(migration: AddTwitterURLToUser.self, database: .psql)
    services.register(migrations)
    ...
}
```

上面完成了数据库迁移的操作，下面更新一个Web应用，以展示

*register.leaf*

```html
...
    <form method="post">
        <div class="form-group">
            <label for="name">Name</label>
            <input type="text" name="name" class="form-control"
            id="name"/>
        </div>
        <div class="form-group">
            <label for="twitterURL">Twitter handle</lable>
            <input type="text" name="twitterURL" class="form-control" id="twitterURL" />
        </div>
...
```

*user.leaf*

```html
<h1>#(user.name)</h1>
<h2>#(user.username)
    #if(user.twitterURL) {
    - #(user.twitterURL)
    }
</h2>
...
```

*WebsiteController.swift*

```swift 
...
struct RegisterData: Content {
    ...
    let twitterURL: String?
}
...
    func registerPostHandler(_ req: Request, data: RegisterData) throws -> Future<Response> {
        ...
        var twitterURL: String?
        if let twitter = data.twitterURL, !twitter.isEmpty {
            twitterURL = twitter
        }
        let user = User(name: data.name, username: data.username, password: password, twitterURL: twitterURL)
        ...
    }
...
```

![register-twitter](/assets/register-twitter.png)

![user-twitter](/assets/user-twitter.png)


# 让类别具有唯一性

*18-12-26-MakeCategoriesUnique.swift*

```swift
import FluentPostgreSQL
import Vapor

struct MakeCategoriesUnique: Migration {
    
    typealias Database = PostgreSQLDatabase
    
    static func prepare(on conn: PostgreSQLConnection) -> Future<Void> {
        return Database.update(Category.self, on: conn) {
            builder in
            builder.unique(on: \.name)
        }
    }
    
    static func revert(on conn: PostgreSQLConnection) -> Future<Void> {
        return Database.update(Category.self, on: conn) { builder in
            builder.deleteUnique(from: \.name)
        }
    }
}
```


*configure.swift*

```swift
public func configure(_ config: inout Config, _ env: inout Environment, _ services: inout Services) throws {
    ...
    migrations.add(migration: MakeCategoriesUnique.self, database: .psql)
    services.register(migrations)
    ...
}
```

# 只在开发和测试环境上创建管理员帐号

*configure.swift*

```swift
public func configure(_ config: inout Config, _ env: inout Environment, _ services: inout Services) throws {
    ...
    switch env {
    case .development, .testing:
        migrations.add(migration: AdminUser.self, database: .psql)
    default:
        break
    }
    migrations.add(migration: AddTwitterURLToUser.self, database: .psql)
    migrations.add(migration: MakeCategoriesUnique.self, database: .psql)
    services.register(migrations)
    ...
}
```