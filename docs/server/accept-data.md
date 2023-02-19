Content协议是Vapor对Codable协议的封装，用来从请求中提取数据。

添加遵守Content协议的结构体InfoData, 它只有一个name字符串成员，Content协议支持请求数据向结构体对象的解码转换。在routes文件中添加下面代码，编译运行

!!! example "POST 请求"
    ```swift hl_lines="20-23 25-27"
    import Vapor

    func routes(_ app: Application) throws {
        app.get { req in
            return "It works!"
        }

        app.get("hello") { req -> String in
            return "Hello, world!"
        }
        
        // Add Routes
        app.get("hello", ":name") { req -> String in
            guard let name = req.parameters.get("name", as: String.self) else {
                return "\(HTTPStatus.notFound)"
            }
            return "Hello, \(name)"
        }
        // ---
        app.post("info") { (req) -> String in
            let info = try req.content.decode(InfoData.self)
            return "Hello, \(info.name)"
        }
    }
    struct InfoData: Content {
        let name: String
    }
    ```

我们使用一个Mac上的应用，名叫[`rested`](https://apps.apple.com/cn/app/rested-simple-http-requests/id421879749?mt=12)模拟`POST`请求，按下面设置测试路由正常。

![post data](assets/post-data.png)