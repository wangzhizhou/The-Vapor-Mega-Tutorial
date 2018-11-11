*UsesTests*
```swift
func testUserCanBeSavedWithAPI() throws {
    
let user = User(name: usersName, username: usersUsername, password: "password")
...
}

func testUsersCanBeRetrievedFromAPI() throws {
    ...
    // 发请求并获得响应
    let users = try app.getResponse(to: usersURI, decodeTo: [User.Public].self)
    ...
}


```

*Models+testable*
```swift
...
import Crypto
extension User {
    static func create(
        name: String = "Luke",
        username: String? = nil,
        on connection: PostgreSQLConnection) throws -> User {
        var createUsername: String
        if let suppliedUsername = username {
            createUsername = suppliedUsername
        } else {
            createUsername = UUID().uuidString
        }
        let password = try BCrypt.hash("password")
        
        let user = User(name: name, username: createUsername, password: password)
        return try user.save(on: connection).wait()
    }
}
...
```

修改后重置docker测试数据库：
```bash
$ docker stop postgres-test
$ docker rm postgres-test
$ docker run --name postgres-test -e POSTGRES_DB=vapor-test \
  -e POSTGRES_USER=vapor -e POSTGRES_PASSWORD=password \
  -p 5433:5432 -d postgres
```

现在运行测试会产生崩溃，因为对那些需要认证的API的调用会失败:

![update-tests-crash-when-run](/assets/update-tests-crash-when-run.png)


*Application+testable.swift*
```swift
...
import Authentication
extension Application {
    ...
    func sendRequest<T>(
        to path: String,
        method: HTTPMethod,
        headers: HTTPHeaders = .init(),
        body: T? = nil,
        loggedInRequest: Bool = false,
        loggedInUser: User? = nil
        ) throws -> Response where T: Content {

        var headers = headers
        if loggedInRequest || loggedInUser != nil {
            let username: String
            if let user = loggedInUser {
                username = user.username
            } else {
                username = "admin"
            }
            let credentials = BasicAuthorization(username: username, password: "password")
            
            var tokenHeaders = HTTPHeaders()
            tokenHeaders.basicAuthorization = credentials
            
            let tokenResponse = try self.sendRequest(to: "/api/users/login", method: .POST, headers:  tokenHeaders)
            
            let token = try tokenResponse.content.syncDecode(Token.self)
            headers.add(name: .authorization, value: "Bearer \(token.token)")
        }
        
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
        headers: HTTPHeaders = .init(),
        loggedInRequest: Bool = false,
        loggedInUser: User? = nil
    ) throws -> Response {
        let emptyContent: EmptyContent? = nil
        return try sendRequest(to: path,
                               method: method,
                               headers: headers,
                               body: emptyContent,
                               loggedInRequest:loggedInRequest,
                               loggedInUser: loggedInUser)
    }
    
    func sendRequest<T>(
        to path: String,
        method: HTTPMethod,
        headers: HTTPHeaders,
        data: T,
        loggedInRequest: Bool = false,
        loggedInUser: User? = nil
        ) throws where T: Content {
        
        _ =  try sendRequest(to: path,
                               method: method,
                               headers: headers,
                               body: data,
                               loggedInRequest: loggedInRequest,
                               loggedInUser: loggedInUser)
    }
    
    
    func getResponse<C, T>(
        to path: String,
        method: HTTPMethod = .GET,
        headers: HTTPHeaders = .init(),
        data: C? = nil,
        decodeTo type: T.Type,
        loggedInRequest: Bool = false,
        loggedInUser: User? = nil
        ) throws -> T where C: Content, T: Decodable {
        
        let response = try self.sendRequest(to: path,
                                            method: method,
                                            headers: headers,
                                            body: data,
                                            loggedInRequest: loggedInRequest,
                                            loggedInUser: loggedInUser)
        // 从响应数据中解析出用户数据，也即从数据库中检索出来的用户数据
        return try response.content.decode(type).wait()
    }
    
    func getResponse<T> (
        to path: String,
        method: HTTPMethod = .GET,
        headers: HTTPHeaders = .init(),
        decodeTo type: T.Type,
        loggedInRequest: Bool = false,
        loggedInUser: User? = nil
    ) throws -> T where T: Decodable {
        let emptyContent: EmptyContent? = nil
        return try self.getResponse(to: path,
                                    method: method,
                                    headers: headers,
                                    data: emptyContent,
                                    decodeTo: type,
                                    loggedInRequest: loggedInRequest,
                                    loggedInUser: loggedInUser)
    }
}
```

*AcronymTests.swift*
```swift
func testAcronymCanBeSavedWithAPI() throws {
    ...
    let receivedAcronym = try app.getResponse(to: acronymsURI, method: .POST, headers: ["Content-Type": "application/json"], data: acronym, decodeTo: Acronym.self, loggedInRequest: true)
    ...
}

func testUpdatingAnAcronym() throws {
    ...
    try app.sendRequest(to: "\(acronymsURI)\(acronym.id!)", method: .PUT, headers: ["Content-Type": "application/json"], data: updatedAcronym, loggedInUser: newUser)
    ...
}

func testDeletingAnAcronym() throws {
    ...
    _ = try app.sendRequest(to: "\(acronymsURI)\(acronym.id!)", method: .DELETE, loggedInRequest: true)
    ...
}

func testGettingAnAcronymsUser() throws {
    ...
    let acronymsUser = try app.getResponse(to: "\(acronymsURI)\(acronym.id!)/user", decodeTo: User.Public.self)
    ...
}
    
func testAcronymsCategories() throws {
    ...
    let categories = try app.getResponse(to: "\(acronymsURI)\(acronym.id!)/categories", decodeTo: [App.Category].self, loggedInRequest: true)
    
    ...
    
    _ = try app.sendRequest(to: "\(acronymsURI)\(acronym.id!)/categories/\(category.id!)", method: .DELETE)
    let newCategories = try app.getResponse(to: "\(acronymsURI)\(acronym.id!)/categories", decodeTo: [App.Category].self, loggedInRequest: true)
    
    ...
}

func testAcronymsCategories() throws {
    ...
    let request1URL = "\(acronymsURI)\(acronym.id!)/categories/\(category.id!)"
    _ = try app.sendRequest(to: request1URL,method: .POST,loggedInRequest: true)
    
    let request2URL = "\(acronymsURI)\(acronym.id!)/categories/\(category2.id!)"
    _ = try app.sendRequest(to: request2URL,method: .POST,loggedInRequest: true)
    
    let categories = try app.getResponse(to: "\(acronymsURI)\(acronym.id!)/categories", decodeTo: [App.Category].self)
    
    XCTAssertEqual(categories.count, 2)
    XCTAssertEqual(categories[0].id, category.id)
    XCTAssertEqual(categories[0].name, category.name)
    XCTAssertEqual(categories[1].id, category2.id)
    XCTAssertEqual(categories[1].name, category2.name)
    
    let request3URL = "\(acronymsURI)\(acronym.id!)/categories/\(category.id!)"
    _ = try app.sendRequest(to: request3URL,method: .DELETE,loggedInRequest: true)
    ...
}
```

*CategoryTests*
```swift
func testCategoryCanBeSavedWithAPI() throws {
    ...
    let receivedCategory = try app.getResponse(to: categoriesURI, method: .POST, headers: ["Content-Type": "application/json"], data: category, decodeTo: Category.self, loggedInRequest:true)
    ...
}

func testGettingACategoriesAcronymsFromTheAPI() throws {
    ...
    let acronym1URL = "/api/acronyms/\(acronym.id!)/categories/\(category.id!)"
    _ = try app.sendRequest(to: acronym1URL, method: .POST,loggedInRequest:true)
    
    let acronym2URL = "/api/acronyms/\(acronym2.id!)/categories/\(category.id!)"
    _ = try app.sendRequest(to: acronym2URL, method: .POST, loggedInRequest: true)
    
    let acronyms = try app.getResponse(to: "\(categoriesURI)\(category.id!)/acronyms", decodeTo: [Acronym].self, loggedInRequest: true
    ...
}


```

*UsesTests*
```swift
func testUsersCanBeRetrievedFromAPI() throws {
    ...
    // 发请求并获得响应
    let users = try app.getResponse(to: usersURI, decodeTo: [User.Public].self)
    ...
    // 考虑到有一个管理员帐户
    XCTAssertEqual(users.count, 3)
    XCTAssertEqual(users[1].name, usersName)
    XCTAssertEqual(users[1].username, usersUsername)
    XCTAssertEqual(users[1].id, user.id)
}

func testUserCanBeSavedWithAPI() throws {
    let user = User(name: usersName, username: usersUsername, password: "password")
    let receivedUser = try app.getResponse(to: usersURI,
                                            method: .POST,
                                            headers: ["Content-Type":"application/json"],
                                            data: user,
                                            decodeTo: User.Public.self,
                                            loggedInRequest: true)
    ...
    let users = try app.getResponse(to: usersURI,
                                decodeTo: [User.Public].self)
    ...
    // 考虑到有一个管理员帐户
    XCTAssertEqual(users.count, 2)
    XCTAssertEqual(users[1].name, usersName)
    XCTAssertEqual(users[1].username, usersUsername)
    XCTAssertEqual(users[1].id, receivedUser.id)
}

func testGettingASingleUserFromTheAPI() throws {
    ...
    let receivedUser = try app.getResponse(to: "\(usersURI)\(user.id!)", decodeTo: User.Public.self)
    ...
}
```

修改完在后一定要跑完单元测试: 

![update-test-cases-successfully](/assets/update-test-cases-successfully.png)