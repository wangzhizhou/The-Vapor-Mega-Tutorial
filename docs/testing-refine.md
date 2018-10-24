我们把所有的代码写在一个函数里面不利于复用，前面我们只写了一个测试用例，但如果要写多个测试用例时会发现，重置数据库这个操作是很多地方会重复使用的，所以我们需要把一些可以重复使用的逻辑抽离出来，以方便写更多的测试。

把数据库重置这一部分抽出到`Application+Testable.swift`中

```swift
import Vapor

extension Application {
    static func testable(envArgs: [String]? = nil) throws -> Application {
        // 创建默认配置对象
        var config = Config.default()
        // 测试环境
        var env = Environment.testing
        // 创建默认服务对象
        var services = Services.default()
        
        if let environmentArgs = envArgs {
            env.arguments = environmentArgs
        }
        // 使用 config和env来配置服务
        try App.configure(&config, &env, &services)
        // 使用config、env和services来初始化一个应用对象
        let app = try Application(config: config, environment: env, services: services)
        // 作一些应用初始化之后要做的工作
        try App.boot(app)
        
        return app
    }
    
    static func reset() throws {
        let revertEnvironmentArgs = ["vapor", "revert", "--all", "-y"]
        try Application.testable(envArgs: revertEnvironmentArgs).asyncRun().wait()
        
        let migrateEnvironmentArgs = ["vapor", "migrate", "-y"]
        try Application.testable(envArgs: migrateEnvironmentArgs).asyncRun().wait()
    }
}

```

抽出这部分逻辑后，之前的那个测试用例重置数据库的部分就可以简化为一句话：

*UserTests.swift*
```swift
...
func testUsersCanBeRetrievedFromAPI() throws {
        // 重置数据库
        try Application.reset()
        // 可测试
        let app = try Application.testable()
        ...
}
...
```

再进一步抽离一些可以复用的逻辑，发现发送请求和获取响应也是可以复用的，逻辑都抽离到`Application+testable.swift`中：

```swift
...
    // 因为模板函数不接收nil，所有定义一个空内容来代替
    struct EmptyContent: Content {}
    
    func sendRequest<T>(
        to path: String,
        method: HTTPMethod,
        headers: HTTPHeaders = .init(),
        body: T? = nil
        ) throws -> Response where T: Content {

        // 创建一个和发送到app的HTTP请求
        let request = HTTPRequest(method: method,
                                  url: URL(string: path)!,
                                  headers: headers)
        let wrappedRequest = Request(http: request, using: self)
        
        if let body = body {
            try wrappedRequest.content.encode(body)
        }
        
        // 因为app本身没有正式运行，所以这里手动进行j响应
        let responder = try self.make(Responder.self)
        return try responder.respond(to: wrappedRequest).wait()
    }
    
    func sendRequest(
        to path: String,
        method: HTTPMethod,
        headers: HTTPHeaders = .init()
    ) throws -> Response {
        let emptyContent: EmptyContent? = nil
        return try sendRequest(to: path,
                               method: method,
                               headers: headers,
                               body: emptyContent)
    }
    
    func sendRequest<T>(
        to path: String,
        method: HTTPMethod,
        headers: HTTPHeaders,
        data: T
        ) throws where T: Content {
        
        _ =  try sendRequest(to: path,
                               method: method,
                               headers: headers,
                               body: data)
    }
    
    
    func getResponse<C, T>(
        to path: String,
        method: HTTPMethod = .GET,
        headers: HTTPHeaders = .init(),
        data: C? = nil,
        decodeTo type: T.Type
        ) throws -> T where C: Content, T: Decodable {
        
        let response = try self.sendRequest(to: path,
                                            method: method,
                                            headers: headers,
                                            body: data)
        // 从响应数据中解析出用户数据，也即从数据库中检索出来的用户数据
        return try response.content.decode(type).wait()
    }
    
    func getResponse<T> (
        to path: String,
        method: HTTPMethod = .GET,
        headers: HTTPHeaders = .init(),
        decodeTo type: T.Type
    ) throws -> T where T: Decodable {
        let emptyContent: EmptyContent? = nil
        return try self.getResponse(to: path,
                                    method: method,
                                    headers: headers,
                                    data: emptyContent,
                                    decodeTo: type)
    }
...
```

将这些逻辑抽到`Application+testable.swift`中后，原来的测试用例简化为： 

```swift
func testUsersCanBeRetrievedFromAPI() throws {
     
     // 重置数据库
     try Application.reset()
     // 可测试
     let app = try Application.testable()
     // 创建一个app到PostgreSQL数据库的连接，表示app使用PostgreSQL数据库连接成功
     let conn = try app.newConnection(to: .psql).wait()

    // 定义测试数据
     let expectedName = "Alice"
     let expectedUsername = "alice"
     // 创建两个用户数据，并保存进数据库中
     let user = User(name: expectedName, username: expectedUsername)
     let savedUser = try user.save(on: conn).wait()
     _ = try User(name: "Luke", username: "lukes").save(on: conn).wait()
     
     // 发请求并获得响应
     let users = try app.getResponse(to: "/api/users", decodeTo: [User].self)
     
     // 进行验证
     XCTAssertEqual(users.count, 2)
     XCTAssertEqual(users[0].name, savedUser.name)
     XCTAssertEqual(users[0].username, savedUser.username)
     XCTAssertEqual(users[0].id, savedUser.id)
     
     // 关闭对数据库的连接，停止使用数据库
     conn.close()
 }
```

再考虑把创建测试数据的部分抽离到`Models+testable.swift`中：
```swift 
import Vapor
import FluentPostgreSQL

extension User {
    static func create(
        name: String = "Luke",
        username: String = "lukes",
        on connection: PostgreSQLConnection) throws -> User {
        let user = User(name: name, username: username)
        return try user.save(on: connection).wait()
    }
}
```

再结合XCTest测试框架的`setUp`和`tearDown`函数，最终测试用例简化如下： 

*UserTests.swit*
```swift
@testable import App
import XCTest
import Vapor
import FluentPostgreSQL

final class UserTests: XCTestCase {

    let usersName = "Alice"
    let usersUsername = "alice"
    let usersURI = "/api/users/"
    var app: Application!
    var conn: PostgreSQLConnection!
    
    override func setUp() {
        // 重置数据库
        try! Application.reset()
        // 可测试
        app = try! Application.testable()
        // 创建一个app到PostgreSQL数据库的连接，表示app使用PostgreSQL数据库连接成功
        conn = try! app.newConnection(to: .psql).wait()
    }
    
    override func tearDown() {
        // 关闭对数据库的连接，停止使用数据库
        conn.close()
    }
    
    func testUsersCanBeRetrievedFromAPI() throws {
        
        // 创建两个测试用户数据
        let user = try User.create(name: usersName,
                                   username: usersUsername,
                                   on: conn)
        _ = try User.create(on: conn)
        
        // 发请求并获得响应
        let users = try app.getResponse(to: usersURI, decodeTo: [User].self)
        
        // 进行验证
        XCTAssertEqual(users.count, 2)
        XCTAssertEqual(users[0].name, usersName)
        XCTAssertEqual(users[0].username, usersUsername)
        XCTAssertEqual(users[0].id, user.id)
    }
}
```

这整个优化过程实在是教科书式的例子。有个这种优化，之后写其它的测试用例，代码会简洁很多。