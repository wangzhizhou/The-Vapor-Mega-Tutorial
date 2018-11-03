*WebsiteController.swift*
```swift
...
struct AllCategoriesContext: Encodable {
    let title = "All Categories"
    let categories: Future<[Category]>
}
struct CategoryConext: Encodable {
    let title: String
    let category: Category
    let acronyms: Future<[Acronym]>
}
...
    router.get("categories", use: allCategoriesHandler)
    router.get("category", Category.parameter, use: categoryHandler)
...
    func allCategoriesHandler(_ req: Request) throws -> Future<View> {
        let categories = Category.query(on: req).all()
        let context = AllCategoriesContext(categories: categories)
        return try req.view().render("allCategories", context)
    }
    func categoryHandler(_ req: Request) throws -> Future<View> {
        return try req.parameters.next(Category.self)
            .flatMap(to: View.self) { category in
                let acronyms = try category.acronyms.query(on: req).all()
                let context = CategoryConext(title: category.name, category: category, acronyms: acronyms)
                return try req.view().render("category", context)
        }
    }
...
```

*allCategories.leaf*
```html
#set("content"){
    <h1>All Categories</h1>
    #if(count(categories) > 0) {
    <table class="table table-bordered table-hover">
        <thead class="thead-light">
            <tr>
                <th>
                    Name
                </th>
            </tr>
        <thead>
        <tbody>
            #for(category in categories) {
                <tr>
                    <td>
                        <a href="/category/#(category.id)">
                            #(category.name)
                        </a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
    } else {
    <h2>There aren't any categories yet!</h2>
    }
}

#embed("base")

```

*category.leaf*
```html
#set("content") {
    <h1>#(category.name)</h1>
    #if(count(acronyms) > 0) {
        <table class="table table-bordered table-hover">
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
                            <a href="/acronyms/#(acronym.id)">
                                #(acronym.short)
                            </a>
                        </td>
                        <td>
                            #(acronym.long)
                        </td>
                    </tr>
                }
            </tbody>
        </table>
    } else {
        <h2>There aren't any acronyms yet!</h2>
    }
}
#embed("base")
```

*base.leaf*
```html
...
<li class="nav-item #if(title=="All Categories"){active}">
    <a href="/categories" class="nav-link">All Categories</a>
</li>
...
```

![all-categories](/assets/all-categories.png)