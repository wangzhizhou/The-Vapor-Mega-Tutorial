创建html页面文件：`Resources/Views/allUsers.leaf`

*allUsers.leaf*
```html
#set("content") {
    <h1>All Users</h1>
    #if(count(users) > 0) {
    <table class="table table-borderd table-hover">
        <thead class="thead-light">
            <tr>
                <th>
                    Username
                </th>
                <th>
                    Name
                </th>
            </tr>
        </thead>
        <tbody>
            #for(user in users) {
            <tr>
                <td>
                    <a href="/users/#(user.id)">#(user.username)</a>
                </td>
                <td>
                    #(user.name)
                </td>
            </tr>
            }
        </tbody>
    </table>
    } else {
    <h2>There aren't any uses yet!</h2>
    }
}

#embed("base")
```

在控制器中添加页面访问逻辑

*WebsiteController.swift*
```swift
...
struct AllUsersContext: Encodable {
    let title: String
    let users: [User]
}
...
    router.get("users", use: allUsersHandler)
...
    func allUsersHandler(_ req: Request) throws -> Future<View> {
        return User.query(on: req).all()
            .flatMap(to: View.self) { users in
                let context = AllUsersContext(title: "All Users", users: users)
                return try req.view().render("allUsers", context)
        }
    }
...
```

最后在web页面添加一个查看所有用户的入口，因为所有的页面都可以进入，所以我们在
base.leaf模板文件中添加这个入口，加在导航列表的位置

*base.leaf*
```html
...
<li class="nav-item #if(title=="All Users"){active}">
    <a href="/users" class="nav-link">All Users</a>
</li>
...
```

![all-users](/assets/all-users.png)