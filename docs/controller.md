当项目很大时，我们不能一条条的写router，因为之后很难进行维护。一般大型项目都要采用一定的设计模式，以确保项目可以有效的进行维护。在Web开发中很流行的一种模式就是MVC，即Model-View-Controller，以Web为例，数据库中的数据结构为模型(Model)，给用户展示的HTML页面为视图(View),控制着数据展示到页面上的那部分逻辑为控制器(Controller)。我们这篇就是讲Controller。

当工程比较大时，一条条的Router组成了请求返回逻辑，处理数据和视图的展示。使用Controller把一条条零散的Router组织成独立的模块，这样有利于划分概念方便后期维护。例如针对数据的CRUD操作就可以使用一个控制器组织在一起。

*routes.swift* 可以简化为：
```swift
import Vapor

/// Register your application's routes here.
public func routes(_ router: Router) throws {
    
    let acronymsController = AcronymsController()
    try router.register(collection: acronymsController)
}
```
分组的所有routes都写入控制器中： 

*AcronymsController.swift*
```swift
import Vapor
import Fluent

struct AcronymsController: RouteCollection {
    func boot(router: Router) throws {
        router.post("api", "acronyms", use: createHandler)
    }
    
    func createHandler(_ req: Request) throws -> Future<Acronym>
    {
        return try req.content.decode(Acronym.self)
            .flatMap(to: Acronym.self) { (acronym) in
                return acronym.save(on: req)
        }
    }
    ...
}
```

但这种方式需要每条route都写明自己的路径，如果控制器中有多条route，它们的路径有很长一部分都是一样的，那么后期如果要同时修改它们的前面相同的部分，对于维护来说将是灾难。所以有了RouteGroup的概念来解决这个问题。

```swift
import Vapor
import Fluent

struct AcronymsController: RouteCollection {
    func boot(router: Router) throws {
        let routeGroup = router.grouped("api", "acronyms")
        
        routeGroup.post(use: createHandler)
        routeGroup.get(use: getAllHandler)
        routeGroup.get(Acronym.parameter, use: getHandler)
        routeGroup.put(Acronym.parameter, use: updateHandler)
        routeGroup.delete(Acronym.parameter, use: deleteHandler)
        routeGroup.get("search", use: searchHandler)
        routeGroup.get("first", use: firstHandler)
        routeGroup.get("sorted", use: sortedHandler)
        
        
    }
    
    func createHandler(_ req: Request) throws -> Future<Acronym>
    {
        return try req.content.decode(Acronym.self)
            .flatMap(to: Acronym.self) { (acronym) in
                return acronym.save(on: req)
        }
    }
    
    func getAllHandler(_ req: Request) throws -> Future<[Acronym]> {
        return Acronym.query(on: req).all()
    }

    func getHandler(_ req: Request) throws -> Future<Acronym> {
        return try req.parameters.next(Acronym.self)
    }

    func updateHandler(_ req: Request) throws -> Future<Acronym> {
        return try flatMap(to: Acronym.self, req.parameters.next(Acronym.self), req.content.decode(Acronym.self)) { (acronym, updateAcronym) -> Future<Acronym> in
            acronym.short = updateAcronym.short
            acronym.long = updateAcronym.long
            
            return acronym.save(on: req)
        }
    }

    func deleteHandler(_ req: Request) throws -> Future<HTTPStatus> {
        return try req.parameters.next(Acronym.self)
            .delete(on: req)
            .transform(to: HTTPStatus.noContent)
    }

    func searchHandler(_ req: Request) throws -> Future<[Acronym]> {
        guard let searchItem = req.query[String.self, at: "term"] else {
            throw Abort(.badRequest)
        }
        return Acronym.query(on: req).group(.or) { (or) in
            or.filter(\.short == searchItem)
            or.filter(\.long == searchItem)
            }
            .all()
    }
    
    func firstHandler(_ req: Request) throws -> Future<Acronym> {
        return Acronym.query(on: req).first().map(to: Acronym.self) { (acronym)  in
            guard let acronym = acronym else {
                throw Abort(.notFound)
            }
            return acronym
        }
    }
    
    func sortedHandler(_ req: Request) throws -> Future<[Acronym]> {
        return Acronym.query(on: req).sort(\.short, .ascending).all()
    }
}

```

另外，还可以改进一个route的使用方式，我们可以不用手动解码请求参数，以createHandler为例：


```swift
...
routeGroup.post(Acronym.self,use: createHandler)
...

func createHandler(_ req: Request, acronym: Acronym) throws -> Future<Acronym> {
    return acronym.save(on: req)
}
```