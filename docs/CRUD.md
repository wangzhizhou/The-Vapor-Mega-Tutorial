RESTful APIs提供了一种方式在应用中调用CRUD函数的方式。通常我们有一个URL对应数据模型，在这个URL基础上配合HTTP不同的请求Method，就可以执行相应的CRUD操作，例如

- POST https://localhost:8080/api/acronyms    创建一个新的缩略语
- GET  https://localhost:8080/api/acronyms    获取所有的缩略语数据
- GET  https://localhost:8080/api/acronyms/**<ID\>**  获取指定ID的缩略语
- PUT  https://localhost:8080/api/acronyms/**<ID\>**  更新指定ID的缩略语
- DELETE https://localhost:8080/api/acronyms/**<ID\>** 删除指定ID的缩略语  

# Create 

*POST https://localhost:8080/api/acronyms    创建一个新的缩略语*

```swift
router.post("api", "acronyms") { (req) -> Future<Acronym> in
    return try req.content.decode(Acronym.self)
        .flatMap(to: Acronym.self) { (acronym) in
            return acronym.save(on: req)
    }
}
```

# Retrieve

*GET  https://localhost:8080/api/acronyms    获取所有的缩略语数据*

```swift
router.get("api", "acronyms") { (req) -> Future<[Acronym]> in
    return Acronym.query(on: req).all()
}
```
*GET  https://localhost:8080/api/acronyms/**<ID\>**  获取指定ID的缩略语*

需要数据模式遵循Parameter协议

*Acronym.swift* 
```
// 使可以作为请求参数
extension Acronym: Parameter {}
```
---

```swift
router.get("api", "acronyms", Acronym.parameter) { (req) -> Future<Acronym> in
    return try req.parameters.next(Acronym.self)
}
```

# Update

*PUT  https://localhost:8080/api/acronyms/**<ID\>**  更新指定ID的缩略语*

```swift
router.put("api","acronyms",Acronym.parameter) { (req) -> Future<Acronym> in
    return try flatMap(to: Acronym.self, req.parameters.next(Acronym.self), req.content.decode(Acronym.self)) { (acronym, updateAcronym) -> Future<Acronym> in
        acronym.short = updateAcronym.short
        acronym.long = updateAcronym.long
        
        return acronym.save(on: req)
    }
}
```
# Delete

*DELETE https://localhost:8080/api/acronyms/**<ID\>** 删除指定ID的缩略语*

```swift
router.delete("api","acronyms", Acronym.parameter) { (req) -> Future<HTTPStatus>in
    return try req.parameters.next(Acronym.self)
    .delete(on: req)
    .transform(to: HTTPStatus.noContent)
}
```