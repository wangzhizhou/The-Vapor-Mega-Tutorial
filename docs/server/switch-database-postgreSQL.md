因为之前图方便，就一直使用的SQLite数据库，它是一个基于文件的数据库系统，当然也有一些功能上的后缺失。

上一篇父子关系试验时发现SQLite不支持外键约束，加了和没加一样。所以决定切换数据到PostgreSQL来验证一下。

我们使用Docker来布置PostgreSQL数据库，首先要安装Docker，这个自己搜官网按照最新安装文档可以完成，这里就不写了, 下面在Shell中运行一段命令，布置一下PostgreSQL的Docker环境：

```bash
docker run --name postgres -e POSTGRES_DB=vapor \
  -e POSTGRES_USER=vapor -e POSTGRES_PASSWORD=password \
  -p 5432:5432 -d postgres
```

!!! note "国内Docker加速器设置，使用DaoCloud加速"
    ```bash tab="Linux"
    curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s    http://f1361db2.m.daocloud.io

    // 该脚本可以将 --registry-mirror 加入到你的 Docker 配置文件 /etc/docker/daemon.json 中。适用于 Ubuntu14.04、Debian、CentOS6 、CentOS7、Fedora、Arch Linux、openSUSE Leap 42.1，其他版本可能有细微不同。更多详情请访问文档。
    ```

    ```bash tab="MacOS"
    // 右键点击桌面顶栏的 docker 图标，选择 Preferences ，在 Daemon 标签（Docker 17.03 之前版本为 Advanced 标签）下的 Registry mirrors 列表中加入下面的镜像地址:
    
    http://f1361db2.m.daocloud.io

    // 点击 Apply & Restart 按钮使设置生效。
    ```

    ```bash tab="Windows"
    // 在桌面右下角状态栏中右键 docker 图标，修改在 Docker Daemon 标签页中的 json ，把下面的地址:

    http://f1361db2.m.daocloud.io

    加到" registry-mirrors"的数组里。点击 Apply 。
    ```

    ![docker-config](/assets/docker-config.png)

# 工程中切换FluentPostgreSQL

*Package.swift*
```swift
// PostgreSQL
.package(url: "https://github.com/vapor/fluent-postgresql", from: "1.0.0")
...
.target(name: "App", dependencies: ["FluentPostgreSQL", "Vapor"]),
```
*configure.swift*
```swift
import FluentPostgreSQL
...
try services.register(FluentPostgreSQLProvider())
...
/// Register the configured SQLite database to the database config.
let databaseConfig = PostgreSQLDatabaseConfig(hostname: "localhost",
                                              port: 5432,
                                              username: "vapor",
                                              database: "vapor",
                                              password: "password")
let database = PostgreSQLDatabase(config: databaseConfig)
 
var databases = DatabasesConfig()
databases.add(database: database, as: .psql)
services.register(databases)

/// Configure migrations
var migrations = MigrationConfig()
migrations.add(model: User.self, database: .psql)
migrations.add(model: Acronym.self, database: .psql)
services.register(migrations)
```

*Acronym.swift*
```swift
import FluentPostgreSQL
...
extension Acronym: PostgreSQLModel {}
...
extension Acronym: Migration {
    static func prepare(on connection: PostgreSQLConnection) -> Future<Void> {
        
        // 创建Acronym在数据库中的表
        return Database.create(self, on: connection) { (builder) in
            
            // 添加Acronym所有属性到表中
            try addProperties(to: builder)
            
            // 这一句添加了Acronym.userID到User.id的外键约束
            builder.reference(from: \.userID, to: \User.id)
        }
    }
}
...
```

*User.swift*
```swift
import FluentPostgreSQL
...
extension User: PostgreSQLUUIDModel {}
```

运行程序，并验证外键约束是否生效。从错误中可以看出，PostgreSQL支持外键约束。

![foreign-key](/assets/foregin-key.png)


!!! info "什么是外键约束"
    外键必须为另一个表中的主键。外键的用途是确保数据的完整性。因为我们在Acronym中定义了外键UserID为User表中的主键id，所以要创建一个Acronym时它对应的User必须是已经存在的才能创建成功，当要删除一个User时，需要先把User下的所有Acronym都删除后，才能删除该User。