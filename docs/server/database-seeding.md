前面我们使用认证中间件，把所有涉及数据修改的API都保护了起来，但也带来一个问题。

试想一下我们刚创建数据库，一个用户都还没有，但这时我们需要创建新用户的时候，API需要我们先以一个合法的用户来登录后才能进行创建用户的操作，但现在数据库里没有任务授权用户的信息，所以认证一定会失败，因此也就无法创建新的用户，数据库完全没有办法使用。

这就需要我们在第一次运行创建数据库的时候往里面塞一个合法用户，也就是我们常见到的管理员帐户。

*User.swift*
```swift
...
struct AdminUser: Migration {
    typealias Database = PostgreSQLDatabase
    
    static func prepare(on conn: PostgreSQLConnection) -> Future<Void> {
        let password = try? BCrypt.hash("password")
        guard let hashPassword = password else {
            fatalError("Failed Create Admin User!")
        }
        
        let user = User(name: "admin", username: "admin", password: hashPassword)
        return user.save(on: conn).transform(to: ())
    }
    
    static func revert(on conn: PostgreSQLConnection) -> Future<Void> {
        return .done(on: conn)
    }
}
```

*configure.swift*
```swift
...
    var migrations = MigrationConfig()
    migrations.add(model: User.self, database: .psql)
    migrations.add(model: Token.self, database: .psql)
    migrations.add(model: Acronym.self, database: .psql)
    migrations.add(model: Category.self, database: .psql)
    migrations.add(model: AcronymCategoryPivot.self, database: .psql)
    migrations.add(migration: AdminUser.self, database: .psql)
    services.register(migrations)
...
```