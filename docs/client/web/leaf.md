`Leaf`是Vapor框架中使用的一种模板语言。模板语言允许传递信息给页面，然后根据所传递的页面信息来动态的生成最终的HTML。模板语言也为我们开发过程中节约代码提供了方法，我们可以使用模板语言来写页面展示代码，其中可以加入各种参数变量，不同页面间也可以复用之前写好的代码块。个性页面显示方式时，有时只需要更改一处地方就可以改变所有页面的显示方式了。我们可以在一个模板中嵌套另一个模板。

我们在之前开发的后端API项目上直接开发Web页面。需要在项目的`Package.swift`文件中添加依赖:

*Package.swift*
```swift
...
.package(url: "https://github.com/vapor/leaf.git", from: "3.0.1")
...
.target(name: "App", dependencies: ["FluentPostgreSQL", "Vapor", "Leaf"]),
```

Leaf模板引擎默认使用目录`Resources/Views`, 所以我们新建一个同名目录来存放相关代码数据和文件。 `mkdir -p Resources/views`

新建一个专门用来返回Web页面的router控制器`WebsiteController.swift`

*WebsiteController.swift*
```swift
import Vapor
import Leaf

struct WebsiteController: RouteCollection {
    
    func boot(router: Router) throws {
        router.get(use: indexHandler)
    }
    
    func indexHandler(_ req: Request) throws -> Future<View> {
        return try req.view().render("index")
    }
}
```

然后在`routes.swift`中注册控制器，使用生效:

```swift
...
let websiteController = WebsiteController()
try router.register(collection: websiteController)
...
```

还要把`Leaf`作为一种服务进行配置：
*configure.swift*
```swift
try services.register(LeafProvider())
config.prefer(LeafRenderer.self, for: ViewRenderer.self)
```

![leaf index](/assets/leaf-index.png)