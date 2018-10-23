测试在软件开发中很重要，在软件开发过程中应该尽量的写单元测试并且将单元测试自动化起来，有样有利于应用的快速迭代。

# 为什么应该写测试

软件测试和软件开发一样古老。现代的服务器应用一天之内可以部署很多次，因此确保每一次的部署都达到预期是很重要的。写测试并让测试用例来保证代码质量是很有前途的事。

在重构代码的时候，测试也可以提供一种保障。手动测试是很没效率的一件事情，当应用变的很大时，手动测试简直就是一种噩梦。如果能够用自动化单元测试的方法保证重要的已有功能正常运行，那么在开发新需求时也会很有信心。

测试同时也可以帮助我们设计代码逻辑。测试驱动开发就是一种在开发前先写测试用例的一种开发方式，这样可以确保很高的代码覆盖率，帮助设计代码和API。

# 使用SPM写测试用例

在Package.swift中定义了一个测试编译目标，它依赖于应用。
*Package.swift*
```swift
targets: [
    ...
    .testTarget(name: "AppTests", dependencies: ["App"])
]
```

测试编译目标要放到`Tests`目录下面, 模板工程的测试编译目标就是在`Tests/AppTests`目录。

使用`vapor xcode -y`生成工程文件后，选择Schema为`*-Package`, 就可以把App和Tests编译目录联系起来，进行测试用例运行。

# 测试User

*UserTests.swift*
```swift 
@testable import App
import XCTest
import Vapor
import FluentPostgreSQL

final class UserTests: XCTestCase {
    func testUsersCanBeRetrievedFromAPI() throws {
        // 定义测试数据
        let expectedName = "Alice"
        let expectedUsername = "alice"
        
        // 创建默认配置对象
        var config = Config.default()
        // 创建默认服务对象
        var services = Services.default()
        // 测试环境
        var env = Environment.testing
        
        // 使用 config和env来配置服务
        try App.configure(&config, &env, &services)
        
        // 使用config、env和services来初始化一个应用对象
        let app = try Application(config: config, environment: env, services: services)
        // 作一些应用初始化之后要做的工作
        try App.boot(app)
        
        // 创建一个app到PostgreSQL数据库的连接，表示app使用PostgreSQL数据库连接成功
        let conn = try app.newConnection(to: .psql).wait()
        
        // 创建两个用户数据，并保存进数据库中
        let user = User(name: expectedName, username: expectedUsername)
        let savedUser = try user.save(on: conn).wait()
        _ = try User(name: "Luke", username: "lukes").save(on: conn).wait()
        
        // 创建一个和发送到app的HTTP请求
        let request = HTTPRequest(method: .GET, url: URL(string: "/api/users")!)
        let wrappedRequest = Request(http: request, using: app)
        
        // 因为app本身没有正式运行，所以这里手动进行j响应
        let responder = try app.make(Responder.self)
        let response = try responder.respond(to: wrappedRequest).wait()
        
        // 从响应数据中解析出用户数据，也即从数据库中检索出来的用户数据
        let data = response.http.body.data
        let users = try JSONDecoder().decode([User].self, from: data!)
        
        // 进行验证
        XCTAssertEqual(users.count, 2)
        XCTAssertEqual(users[0].name, savedUser.name)
        XCTAssertEqual(users[0].username, savedUser.username)
        XCTAssertEqual(users[0].id, savedUser.id)
        
        // 关闭对数据库的连接，停止使用数据库
        conn.close()
    }
    
}
```
为了把数据库在生产环境和测试环境区分开，所以App.configure函数中根据环境，进行了不同的数据库配置操作：

*configure.swift*
```swift
...
let databaseName: String
let databasePort: Int

if(env == .testing) {
    databaseName = "vapor-test"
    databasePort = 5433
} else {
    databaseName = Environment.get("DATABASE_NAME") ?? "vapor"
    databasePort = 5432
}

let username = Environment.get("DATABASE_USERNAME") ?? "vapor"
let password = Environment.get("DATABASE_PASSWORD") ?? "password"
let hostname = Environment.get("DATABASE_HOSTNAME") ?? "localhost"

/// Register the configured SQLite database to the database config.
let databaseConfig = PostgreSQLDatabaseConfig(hostname: hostname,
                                              port: databasePort,
                                              username: username,
                                              database: databaseName,
                                              password: password)
let database = PostgreSQLDatabase(config: databaseConfig)

var databases = DatabasesConfig()
databases.add(database: database, as: .psql)
services.register(databases)
...
// 添加Fluent命令到CommandConfig中，可以使用字符串作为标识符执行"revert"命令和"migrate"命令
var commandConfig  = CommandConfig.default()
commandConfig.useFluentCommands()
services.register(commandConfig)
```

因为我们区分环境，配置了不同的数据库，所以在docker中需要启动一个专门用来测试的数据库：

```bash
docker run \
--name postgres-test \
-e POSTGRES_DB=vapor-test \
-e POSTGRES_USER=vapor \
-e POSTGRES_PASSWORD=password \
-p 5433:5432 \
-d postgres
```

第一次运行测试可以通过，第二次运行就失败，这是因为，第一次运行时往测试数据库存入两个数据，第二次运行时又往测试数据库中存入两个数据，本身测试数据库现在有4条用户数据，不满足第一条断言`XCTAssertEqual(users.count, 2)`, 所以第二次测试失败了。这就是说，每次运行测试时都需要重置数据库。

重置数据库代码如下, 把它放在`testUsersCanBeRetrievedFromAPI`最前面：
```swift 
// 重置数据库
var revertConfig = Config.default()
var revertEnv = Environment.testing
let revertEnvironmentArgs = ["vapor", "revert", "--all", "-y"]
revertEnv.arguments = revertEnvironmentArgs
var revertServices = Services.default()

try App.configure(&revertConfig, &revertEnv, &revertServices)
let revertApp = try Application(config: revertConfig, environment: revertEn services: revertServices)
try App.boot(revertApp)
try revertApp.asyncRun().wait()


let migrateEnvironmentArgs = ["vapor", "migrate", "-y"]
var migrateConfig = Config.default()
var migrateServices = Services.default()
var migrateEnv = Environment.testing
migrateEnv.arguments = migrateEnvironmentArgs
try App.configure(&migrateConfig, &migrateEnv, &migrateServices)
let migrateApp = try Application(config: migrateConfig, environmentmigrateEnv, services: migrateServices)
try App.boot(migrateApp)
try migrateApp.asyncRun().wait()
```

有了重置数据库这个操作，我们可以多次运行测试用例，发现都可以通过了。