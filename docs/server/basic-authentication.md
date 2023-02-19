HTTP基本认证过程是把认证信息放在请求的认证头中。认证信息的生成过程是把用户名和密码合并后进行base64加密后生成token。如果用户名是User，密码是password，那么认证信息就是：base64(user:password) = `dXNlcjpwYXNzd29yZAo=`, 认证头就是：
```
Authorization: Basic dXNlcjpwYXNzd29yZAo=
```

Vapor提供了一个包专门处理各种认证，包括这种基本认证。需要添加一个依赖:

*Package.swift*
```swift
...
.package(url: "https://github.com/vapor/auth.git", from: "2.0.1"),
...
.target(name: "App", dependencies: ["FluentPostgreSQL", "Authentication", "Vapor", "Leaf"]),
...
```

重新生成Xcode工程: `vapor xcode -y`

*User.swift*
```swift
...
import Authentication
...

extension User: BasicAuthenticatable {
    static var usernameKey: WritableKeyPath<User, String> = \.username
    static var passwordKey: WritableKeyPath<User, String> = \.password
}
```

这种User数据支持了基本认证，指定了用户名和密码对应结构中的属性名称

*AcronymsController.swift*
```swift
...
import Authentication
...
struct AcronymsController: RouteCollection {
    func boot(router: Router) throws {
        let routeGroup = router.grouped("api", "acronyms")
        
        let basicAuthenticationMiddleware = User.basicAuthMiddleware(using: BCryptDigest())
        let guardAuthMiddleware = User.guardAuthMiddleware()
        let protectd = routeGroup.grouped(basicAuthenticationMiddleware, guardAuthMiddleware)
        protectd.post(Acronym.self, use: createHandler)
        // routeGroup.post(Acronym.self, use: createHandler) 
        ...
    }
    ...
}
```

!!! info "中间件Middleware"
    中间件可以用来拦截请求和返回数据，在这里，基本认证中间件拦截了请求数据，使用基本认证方法检验发起请求的用户是否合法，保证认证中间件确保认证通过的用户请求继续进行。

上面的中间件正常工作还需要在项目配置中注册中间件服务提供者。

*configure.swift*
```swift
...
import Authentication
...
public func configure(_ config: inout Config, _ env: inout Environment, _ services: inout Services) throws {
    /// Register providers first
    try services.register(FluentPostgreSQLProvider())
    try services.register(AuthenticationProvider())
    ...
}
```

运行工程，并使用Rested工具进行检验, 因为没有设置认证信息头，所以认证失败不能创建缩略语

![user not authentication](assets/user-not-authentication.png)

请求时设置认证信息头
![user-authentication-with-user-password](assets/user-authentication-with-user-password.png)

用户认证通过，创建缩略语成功
![user-authentication-passed](assets/user-authentication-passed.png)