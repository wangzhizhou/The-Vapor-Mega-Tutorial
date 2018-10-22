兄弟关系是用来描述两个数据模型间相互链接关系的。也叫作多对多关系。在兄弟关系间不存在约束。我们这一部分要给缩略语添加类别，那么一个缩略语可以属于一个或多个类别，一个类别也可以包含一个或多个缩略语，这就是一种兄弟关系，即多对多关系。

之前的父子关系中，我们在Acronym中添加了User的主键作为外键，在兄弟关系中如果使用这种方式实现，会有效率问题。如果每一个类别下有多个缩略语，如果要查询某个缩略语属于多少个类别，就需要查询所有的类别。如果每个缩略语下有它属于的所有类别，那个查询某个类别下有多少个缩略语就需要查询所有缩略语，这很明显有效率问题。

所以我们需要抽象出另外一种数据模型来记录类别和缩略语之间的对应关系。这个抽象出的数据模型叫作支点(pivot)。首先创建一个支点数据模型文件：

*AcronymCategoryPivot.swift*
```swift
//
//  AcronymCategoryPivot.swift
//  App
//
//  Created by joker on 2018/10/22.
//

import FluentPostgreSQL
import Foundation

final class AcronymCategoryPivot: PostgreSQLUUIDPivot, ModifiablePivot {

    var id: UUID?
    var acronymID: Acronym.ID
    var categoryID: Category.ID
    
    typealias Left = Acronym
    typealias Right = Category
    
    
    static let leftIDKey: LeftIDKey = \.acronymID
    static let rightIDKey: RightIDKey = \.categoryID
    
    init(_ acronym: Acronym, _ category: Category) throws {
        self.acronymID = try acronym.requireID()
        self.categoryID = try category.requireID()
    }
}

extension AcronymCategoryPivot: Migration {}

```

*configure.swift*
```swift
migrations.add(model: AcronymCategoryPivot.self, database: .psql)
```

# Acronym侧

*Aronym.swift*
```swift
// 获取兄弟关系数据
extension Acronym {
    var categories: Siblings<Acronym, Category, AcronymCategoryPivot> {
        return siblings()
    }
}
```

*AcronymsController.swift*
```swift
routeGroup.post(Acronym.parameter,"categories", Category.parameter, use: addCategoriesHandler)
routeGroup.delete(Acronym.parameter, "categories", Category.parameter, use: removeCategoriesHandler)
routeGroup.get(Acronym.parameter, "categories", use: getCagtegoriesHandler)
...
func addCategoriesHandler(_ req: Request) throws -> Future<HTTPStatus> {
    return try flatMap(to: HTTPStatus.self,
                       req.parameters.next(Acronym.self),
                       req.parameters.next(Category.self)) { (acronym, category) in
                        return acronym
                            .categories
                            .attach(category, on: req)
                            .transform(to: HTTPStatus.created)
    }
}

func removeCategoriesHandler(_ req: Request) throws -> Future<HTTPStatus> {
    return try flatMap(to: HTTPStatus.self, req.parameters.next(Acronym.self), req.parameters.net(Category.self)) { (acronym, category) in
        return  acronym.categories.detach(category, on: req).transform(to: HTTPStatus.noContent)
    }
}

func getCagtegoriesHandler(_ req: Request) throws -> Future<[Category]> {
    return try req.parameters.next(Acronym.self).flatMap(to: [Category].self) { (acronym) in
        return try acronym.categories.query(on: req).all()
    }
}
```

# Category侧

*Category.swift*
```swift
extension Category {
    var acronyms: Siblings<Category, Acronym, AcronymCategoryPivot> {
        return siblings()
    }
}
```

*CategoriesController.swift*
```swift
categoriesRoute.get(Category.parameter,"acronyms",use: getAcronymsHandler)
...
func getAcronymsHandler(_ req: Request) throws -> Future<[Acronym]> {
    return try req.parameters.next(Category.self).flatMap(to: [Acronym].self) { (category) in
        return try category.acronyms.query(on: req).all()
    }
}
```

# AcronymCategoryPivot侧外键约束添加

*AcronymCategoryPivot.swift*
```swift
...
extension AcronymCategoryPivot: Migration {
    static func prepare(on connection: PostgreSQLDatabase.Connection) -> Future<Void> {
        return Database.create(self, on: connection) { (builder) in
            try addProperties(to: builder)
            
            builder.reference(from: \.acronymID, to: \Acronym.id, onDelete: .cascade)
            builder.reference(from: \.categoryID, to: \Category.id, onDelete: .cascade)
        }
    }
}
...
```

# 重置数据库

```bash
$ docker stop postgresql
$ docker rm postgresql
$ docker run --name postgres -e POSTGRES_DB=vapor \
  -e POSTGRES_USER=vapor -e POSTGRES_PASSWORD=password \
  -p 5432:5432 -d postgres
```