需要`import Fluent `

# Filter

*http://localhost:8080/api/acronyms/search?term={searchItem}* 

```swift
router.get("api", "acronyms", "search") { (req) -> Future<[Acronym]> in
    guard let searchItem = req.query[String.self, at: "term"] else {
        throw Abort(.badRequest)
    }
    return Acronym.query(on: req)
    .filter(\.short == searchItem)
    .all()
}
```

```swift
router.get("api", "acronyms", "search") { (req) -> Future<[Acronym]> in
    guard let searchItem = req.query[String.self, at: "term"] else {
        throw Abort(.badRequest)
    }
    return Acronym.query(on: req).group(.or) { (or) in
        or.filter(\.short == searchItem)
        or.filter(\.long == searchItem)
    }
    .all()
}
```

# First

```swift
router.get("api","acronyms","first") { (req) -> Future<Acronym> in
    return Acronym.query(on: req).first().map(to: Acronym.self) { (acronym)  in
        guard let acronym = acronym else {
            throw Abort(.notFound)
        }
        return acronym
    }
}
```

或者更简单一点就是： 

```swift
func firstHandler(_ req: Request) throws -> Future<Acronym> {
    return Acronym.query(on: req).first().unwrap(or: Abort(.notFound))
}
```

# Sort

```swift
router.get("api","acronyms","sorted") { (req) -> Future<[Acronym]> in
    return Acronym.query(on: req).sort(\.short, .ascending).all()
}
```