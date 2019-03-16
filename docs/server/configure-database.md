Vapor原生提供了三种数据库的驱动:`SQLite`、`MySQL`、`PostgreSQL`

数据库分为两种：`关系型数据库(SQL)`和`非关系型数据库(NoSQL)`。关系型数据库适合存放结构化的数据，目前Vapor只支持关系型数据库。但不久的将来也会支持非关系型数据库。

关系型数据库面临的问题是已经存入数据库的结构在后期可能需要变更结构，非关系型数据库就是为了解决这个问题。

# SQLite

一个简单的基于文件的关系型数据库系统，设计目标是嵌入到应用中使用，对于单进程应用很有用。它依靠文件锁保证数据库完整性，所以不适合大量写入操作的应用，也不能跨服务器使用。它适合在开发原型和测试时使用。

# MySQL

一个开源关系型数据库，流行于LAMP应用栈中，它很容易使用同时被大多数云提供商支持。

# PostgreSQL

一个开源关系数据库。主要设计目标是商用，扩展性和标准化是它的核心目标。

# Docker

我们使用Docker在本地布置数据库，它有别于虚拟机，很方便。MySQL和PostgreSQL可以使用Docker部署在本地。


# 只使用SQLite可以简化开发，适用于原型开发期

为了简化数据库配置，我们使用SQLite使用本地文件存放数据。配置数据库

*configure.swift*
```swift
   /// Register providers first
    try services.register(FluentSQLiteProvider())

    // Configure a SQLite database
    let sqlite = try SQLiteDatabase(storage: .file(path: "db.sqlite"))
    
    /// Register the configured SQLite database to the database config.
    var databases = DatabasesConfig()
    databases.add(database: sqlite, as: .sqlite)
    services.register(databases)

    /// Configure migrations
    var migrations = MigrationConfig()
    migrations.add(model: Acronym.self, database: .sqlite)
    services.register(migrations)
```

*Package.swift*
```
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "TILApp",
    dependencies: [
        // 💧 A server-side Swift web framework.
        .package(url: "https://github.com/vapor/vapor.git", from: "3.0.0"),

        // 🔵 Swift ORM (queries, models, relations, etc) built on SQLite 3.
        .package(url: "https://github.com/vapor/fluent-sqlite.git", from: "3.0.0")
    ],
    targets: [
        .target(name: "App", dependencies: ["FluentSQLite", "Vapor"]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"])
    ]
```

数据库的配置主要在`configure.swift`文件中完成。首先注册`FluentSQLiteProvider`服务，这个服务是通过`Fluent`这个ORM中间层来对SQLite数据库进行操作的服务，让开发者可以不用理会具体的数据库。

然后创建数据库，因为`Vapor`支持在同一个应用内使用多个数据库实例，所以添加数据库实例的方式是先创建一个数据库配置(DatabasesConfig)，记录所有用到的数据库实例，然后去统一注册进服务里。

最后把数据库要存放的数据模型和对应要存放的数据库类型绑定，统一使用`Migration`服务进行注册使用。