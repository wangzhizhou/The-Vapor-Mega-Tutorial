认证(Authentication)是验证某人合法的过程。这个过程不同于授权(Authorization)，授权是赋予用户执行某个动作的权力。认证通常是使用`用户名`和`密码`的组合来进行，TIL应用也不例外。

在User数据模型中添加password属性，并修改初始化方法：

*User.swift*
```swift 
final class User: Codable {
    var id: UUID?
    var name: String
    var username: String
    var password: String
    
    init(name: String, username: String, password: String) {
        self.name = name
        self.username = username
        self.password = password
    }
}
```

存储密码有一个原则，就是不能明文的方式存储。Vapor提供了工业级的算法`BCrypt`，用来对密码明文进行哈希计算。哈希的过程是单向的，密码明文能够生成哈希值，但从生成的哈希值无法反向生成密码明文或者说复原密码明文的代码太大不可接受。`BCrypt`在哈希的过程中会自动加盐，使从哈希值复原密码明文几乎不可能。同时`BCrypt`提供验证密码明文和对应密码哈希值的功能。所以我们在保存一个新创建的用户之前，要对用户注册时提供的密码明文进行哈希后再进行存储。加密算法都放在`Crypto`库中。

*UserController.swift*
```swift
...
import Crypto

struct UsersController: RouteCollection {
    ...
    func createHandler(_ req: Request, user: User) throws -> Future<User> {
        user.password = try BCrypt.hash(user.password)
        return user.save(on: req)
    }
    ...
```
这样在用户创建时，用户的密码是以哈希值的方式被存放，即使未来被其它人获得了服务端的用户信息，也很难破解出用户的密码明文来。

# 用户名唯一性
目前来看，我们对同名用户的创建没有作限制，但之后用户在登录时需要通过用户名和密码过行验证，如果有同名用户存在，我们在同名用户登录时就无法决定去验证哪一个用户，所以必须保证用户名的唯一性。

*User.swift*
```swift
...
extension User: Migration {
    static func prepare(on conn: PostgreSQLConnection) -> Future<Void> {
        return Database.create(self, on: conn, closure: { (builder) in
            try addProperties(to: builder)
            builder.unique(on: \.username)
        })
    }
}
...
```
这种操作类似于添加外键约束，如果有同名用户已经创建了，那个新同名用户就会创建不成功。

因为我们的数据库表字段变更和约束添加，需要我们重置一次数据库，让变动生效。`Option + Run`进行Schema设置页，选择参数选择页，添加运行时参数如下：

![revert database](/assets/xcode-revert-all-yes.png)

添加参数并打勾运行一次相当于运行命令`vapor run revert --all --yes`可以重置本地数据库。

![revert local database](/assets/revert-local-database.png)

重置本地数据库之后，再去运行工程时就要把Schema中的参数`revert --all --yes`选择取消勾选了。这样新的数据库表关系就会生效了。我们使用`Rested`应用来创建几个用户：

![create user](/assets/create-user.png)

![create exist user](/assets/create-exist-user.png)


从图中可以看出，我们的请求返回数据中包含了用户的密码哈希值，这显然是不安全的，我们在请求返回数据中应该不包含密码相关的信息。这需要我们在返回用户数据前作一个转换。

*User.swift*
```swift
...
final class User: Codable {
    ...
    
    final class Public: Codable {
        var id: UUID?
        var name: String
        var username: String
        init(id: UUID?, name: String, username: String) {
            self.id = id
            self.name = name
            self.username = username
        }
    }
}
...
extension User.Public: Content {}
...
extension User {
    func convertToPublic() -> User.Public {
        return User.Public(id: self.id, name: self.name, username: self.username)
    }
}
extension Future where T: User {
    
    func convertToPublic() -> Future<User.Public> {
        return self.map(to: User.Public.self) { (user) -> User.Public in
            return user.convertToPublic()
        }
    }
}
```

*UserController.swift*
```swift
import Vapor
import Crypto

struct UsersController: RouteCollection {
    ...
    func createHandler(_ req: Request, user: User) throws -> Future<User.Public> {
        user.password = try BCrypt.hash(user.password)
        return user.save(on: req).convertToPublic()
    }
    
    func getAllHandler(_ req: Request) throws -> Future<[User.Public]> {
        return User.query(on: req).decode(data: User.Public.self).all()
    }
    
    func getHandler(_ req: Request) throws -> Future<User.Public> {
        return try req.parameters.next(User.self).convertToPublic()
    }
    
    func updateHandler(_ req: Request) throws -> Future<User.Public> {
        return try flatMap(to: User.Public.self, req.parameters.next(User.self), req.content.decode(User.self)) { (user, updatedUser) -> Future<User.Public> in
            
            user.name = updatedUser.name
            user.username = updatedUser.username
            
            return user.save(on: req).convertToPublic()
        }
    }
    ...
}
```

![create user no password hash](/assets/create-user-no-password-hash.png)

![get all user no password hash](/assets/get-all-users-no-password-hash.png)

![update user with error](/assets/update-user-with-error.png)

现在所有的请求返回数据中都不包含密码相关信息了。