前面我们建立起一个可以创建缩略语、类别的web应用，当我们使用表单填写数据时，要保证填写数据符合一定的规则，如果不符合规则，就显示录入错误提示，这个过程叫作数据有效性验证。

首先我们来创建用户注册界面。

*register.leaf*
```html
#set("content") {
    <h1>#(title)</h1>
    <form method="post">
        <div class="form-group">
            <label for="name">Name</label>
            <input type="text" name="name" class="form-control"
            id="name"/>
        </div>
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
        <div class="form-group">
            <label for="confirmPassword">Confirm Password</label>
            <input type="password" name="confirmPassword"
            class="form-control" id="confirmPassword"/>
        </div>
        <button type="submit" class="btn btn-primary">
            Register
        </button>
    </form>
}
#embed("base")
```

*WebsiteController.swift*

```swift
struct RegisterContext: Encodable {
    let title = "Register"
}

struct RegisterData: Content {
    let name: String
    let username: String
    let password: String
    let confirmPassword: String
}
func boot(router: Router) throws {
        ...
        authSessionRoutes.get("register", use: registerHandler)
        authSessionRoutes.post(RegisterData.self, at: "register", use: registerPostHandler)
        ...
    }
    
    ...
    func registerHandler(_ req: Request) throws -> Future<View> {
        let context = RegisterContext()
        return try req.view().render("register", context)
    }
    func registerPostHandler(_ req: Request, data: RegisterData) throws -> Future<Response> {
        let password = try BCrypt.hash(data.password)
        let user = User(name: data.name, username: data.username, password: password)
        
        return user.save(on: req).map(to: Response.self) { user in
            try req.authenticateSession(user)
            return req.redirect(to: "/")
        }
    }    
    ...
```


*base.leaf*
```html
...
        #if(!userLoggedIn) {
            <li class="nav-item #if(title == "Register"){active}">
                <a href="/register" class="nav-link">Register</a>
            </li>
        }
    </ul>
...
```

![register-user](/assets/register-user.png)


# 注册用户时的数据用效性验证


## 基本数据有效性验证

*WebsiteController.swift*
```swift
extension RegisterData: Validatable, Reflectable {
    static func validations() throws -> Validations<RegisterData> {
        var validations = Validations(RegisterData.self)
        try validations.add(\.name, .ascii)
        try validations.add(\.username, .alphanumeric && .count(3...))
        try validations.add(\.password, .count(8...))
        return validations
    }
}
...
    func registerPostHandler(_ req: Request, data: RegisterData) throws -> Future<Response> {
        do {
            try data.validate()
        } catch {
            return req.future(req.redirect(to: "register"))
        }
        ...
    }
```

## 自定义数据有效性验证

前面添加的数据有效性验证都是针对单个字段的，但是两次输入密码的一致性验证vapor并没有提供内建的方法，我们需要使用自定义有效性验证

*WebsiteController.swift*
```swift 
extension RegisterData: Validatable, Reflectable {
    static func validations() throws -> Validations<RegisterData> {
        ...
        validations.add("passwords match", { (model) in
            guard model.password == model.confirmPassword else {
                throw BasicValidationError("passwords don't match")
            }
        })
        return validations
    }
}
```


## 注册页面显示错误信息

前面完成了数据有性效验证，但发生错误时没有途径通知用户修正自己的行为，所以需要在出错时显示错误信息，以提示用户

*register.leaf*
```html
#set("content") {
    <h1>#(title)</h1>
    #if(message) {
        <div class="alert alert-danger" role="alert">
            Please fix the following errors:<br />
            #(message)
        </div>
    }
    ...
```

*WebsiteController.swift*
```swift
struct RegisterContext: Encodable {
    let title = "Register"
    let message: String?
    
    init(message: String? = nil) {
        self.message = message
    }
}
...
    func registerHandler(_ req: Request) throws -> Future<View> {
        let context: RegisterContext
        if let message = req.query[String.self, at: "message"] {
            context = RegisterContext(message: message)
        } else {
            context = RegisterContext()
        }
        ...
    }

 func registerPostHandler(_ req: Request, data: RegisterData) throws -> Future<Response> {
        do {
            try data.validate()
        } catch(let error) {
            let redirect: String
            if let error = error as? ValidationError,
                let message = error.reason.addingPercentEncoding(
                    withAllowedCharacters: .urlQueryAllowed) {
                redirect = "/register?message=\(message)"
            } else {
                redirect = "/register?message=Unknown+error"
            }
            return req.future(req.redirect(to: redirect))
        }
        ...
        }
    }
```

![register validation](/assets/register-validation.png)