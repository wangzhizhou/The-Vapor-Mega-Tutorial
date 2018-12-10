Cookie是服务器应用发送给浏览器存放在用户计算机上的小块信息，因为对于浏览器来说，在请求网页时是不能设置自定义头部信息的，所以浏览器在访问服务端应用时会在请求中带上服务端应用发给它的cookie信息。Session能够在多次不同的请求中保存状态信息，在用户验证过程中，服务端会给用户创建一个会话，并使用唯一ID来标识这个会话，发送给用户的cookie中也包含这个标识会话的唯一ID，当用户向服务端应用发送请求时会带上cookie信息，所以能过唯一ID就可以识别出上哪个会话了。

Vapor使用中间件来管理会话

*configure.swift*
```swift
...
middlewares.use(SessionsMiddleware.self)
...
config.prefer(MemoryKeyedCache.self, for: KeyedCache.self)
```

*User.swift*
```swift
...
extension User: PasswordAuthenticatable {}
extension User: SessionAuthenticatable {}
```

使用会话中间件，并配置了键值存储服务，让用户遵循密码验证和会话验证协议。

要实现用户登录需要提供两个api, 一个用来显示登录页面，另一个用来接收用户登录信息。


*WebsiteController.swift*

```swift

struct LoginContext: Encodable {
    let title = "Log In"
    let loginError: Bool
    
    init(loginError: Bool = false) {
        self.loginError = loginError
    }
}

...

    func loginHandler(_ req: Request) throws -> Future<View> {
        let context: LoginContext
        
        if req.query[Bool.self, at: "error"] != nil {
            context = LoginContext(loginError: true)
        } else {
            context = LoginContext()
        }
        
        return try req.view().render("login", context)
    }
    ...
```

页面api和页面参数结构定义好了，我们需要创建页面模板文件来渲染页面显示。

```login.leaf
#set("content") {

<h1>#(title)</h1>

#if(loginError) {
    <div class="alert alert-danger" role="alert">
        User authentication error. Either your username or password
        was invalid.
    </div>
}
<form method="post">
    <div class="form-group">
        <label for="username">Username</label>
        <input type="text" name="username" class="form-control"
        id="username"/>
    </div>
    <div class="form-group">
        <label for="password">Password</label>
        <input type="password" name="password" class="form-control"
        id="password"/>
    </div>
    <button type="submit" class="btn btn-primary">Log In</button>
</form>
}
#embed("base")
```

*WebsiteController.swift*

```swift
...
import Authentication
...
struct LoginPostData: Content {
    let username: String
    let password: String
}
...
    func boot(router: Router) throws {
        ...
        router.get("login", use: loginHandler)
        router.post(LoginPostData.self, at: "login", use: loginPostHandler)
    }
    func loginPostHandler(_ req: Request, userData: LoginPostData) throws -> Future<Response> {
        return User.authenticate(username: userData.username, password: userData.password, using: BCryptDigest(), on: req).map(to: Response.self) {
            user in
            
            guard let user = user else {
                return req.redirect(to: "/login?error")
            }
            
            try req.authenticateSession(user)
            return req.redirect(to: "/")
        }
    }
```

![Log In Error](/assets/login_error.png)