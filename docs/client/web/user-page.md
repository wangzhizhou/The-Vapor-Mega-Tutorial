添加一个leaf文件，用来显示用户想关的信息
*user.leaf*

```swift
#set("content") {
<h1>#(user.name)</h1>
<h2>#(user.username)</h2>

#if(count(acronyms) > 0) {
    <table class = "table table-bordered table-hover">
        <thead class="thead-light">
            <tr>
                <th>
                    Short
                </th>
                <th>
                    Long
                </th>
            </tr>
        </thead>
        <tbody>
            #for(acronym in acronyms) {
                <tr>
                    <td>
                        <a href="/acronyms/#(acronym.id)">#(acronym.short)</a>
                    </td>
                    <td>#(acronym.long)</td>
                </tr>
            }
        </tbody>
        } else {
        <h2>There aren't any acronyms yet!</h2>
        }
}

#embed("base")

```


然后在控制器里添加访问关系： 
*WebsiteController.swift*
```swift
...
struct UserContext: Encodable {
    let title: String
    let user: User
    let acronyms: [Acronym]
}
...
    router.get("users", User.parameter, use: userHandler)
...
    func userHandler(_ req: Request) throws -> Future<View> {
        return try req.parameters.next(User.self).flatMap(to: View.self) { user in
            return try user.acronyms.query(on: req).all()
                .flatMap(to: View.self) { acronyms in
                    let context = UserContext(title: user.name, user: user, acronyms: acronyms)
                    return try req.view().render("user", context)
            }
        }
    }
...
```


最后在`acronym.leaf`文件中添加跳转到用户页面的路径

*acronym.leaf*
```html
...
<p>Created by <a href="/users/#(user.id)/">#(user.name)</a></p>
...
```

![user-page](/assets/user-page.png)