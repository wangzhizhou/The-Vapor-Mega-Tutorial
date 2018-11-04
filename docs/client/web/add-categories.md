*Category.swift*

给Categor模型添加一个扩展方法，用来给缩略语添加类别标签
```swift
extension Category {
    ...

    static func addCategory(_ name: String,
                            to acronym: Acronym,
                            on req: Request
        ) throws -> Future<Void> {
        return Category.query(on: req).filter(\.name == name).first()
            .flatMap(to: Void.self) { foundCategory in
                if let existingCategory = foundCategory {
                    return acronym.categories.attach(existingCategory, on: req)
                    .transform(to: ())
                }
                else {
                    let category = Category(name: name)
                    return category.save(on: req).flatMap(to: Void.self) { savedCategory in
                        return acronym.categories.attach(savedCategory, on: req).transform(to: ())
                        
                    }
                }
        }
    }
}
```


*WebsiteController.swift*
```swift
...
struct CreateAcronymData: Content {
    let userID: User.ID
    let short: String
    let long: String
    let categories: [String]?
}
...
router.post(CreateAcronymData.self, at: "acronyms", "create", use: createAcronymPostHandler)
...
func createAcronymPostHandler(_ req: Request, data: CreateAcronymData) throws -> Future<Response> {
        
        let acronym = Acronym(short: data.short, long: data.long, userID: data.userID)
        
        
        return acronym.save(on: req).flatMap(to: Response.self) { acronym in
            guard let id = acronym.id else {
                throw Abort(HTTPStatus.internalServerError)
            }
            var categorySaves: [Future<Void>] = []
            for category in data.categories ?? [] {
                try categorySaves.append(Category.addCategory(category, to: acronym, on: req))
            }
            let redirect = req.redirect(to: "/acronyms/\(id)")
            return categorySaves.flatten(on: req).transform(to: redirect)
        }
    }
...
```

*createAcronym.leaf*

`categories[]`可发送经url编码后的categories数组数据， `multiple`允许多选
```html
    ...
        <div class="form-group">
            <label for="categories">Categories</label>
            <select name="categories[]" class="form-control" id="categories" placeholder="Categories" multiple="multiple">
            </select>
        </div>
        <button type="submit" class = "btn btn-primary">
            #if(editing) { Update } else { Submit }
        </button>
    </form>
}
...
```

![empty-add-categories](/assets/empty-add-categories.png)

因为我们目前的缩略语还没有相关类别，所以显示是空的，也不好看，所以我们使用一个叫作[`Select2`](https://cdnjs.com/libraries/select2)的javascript库来美化一下。因为jQuery的slim版本不包含`select2`需要用到的函数，所以换了一个版本的jQuery。同时本地写了一个js，也包含在模板文件中了。

*base.leaf*
```html
...
<head>
    ...
    #if(title=="Create An Acronym") {
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/select2/4.0.5/css/select2.css" />
    }
    #if(title=="Edit Acronym"){
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/select2/4.0.5/css/select2.css" />
    }
<title>#(title) | Acronyms</title>
</head>
...
<!-- Optional JavaScript -->
<!-- jQuery first, then Popper.js, then Bootstrap JS -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>

#if(title=="Create An Acronym") {
    <script src="https://cdnjs.cloudflare.com/ajax/libs/select2/4.0.5/js/select2.min.js"></script>
    <script src="/scripts/createAcronym.js"></script>
}
#if(title == "Edit Acronym"){
    <script src="https://cdnjs.cloudflare.com/ajax/libs/select2/4.0.5/js/select2.min.js"></script>
    <script src="/scripts/createAcronym.js"></script>
}
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
</body>
```

*Public/scripts/createAcronym.js*
```js
$.ajax({
       url: "/api/categories/",
       type: "GET",
       contentType: "application/json; charset=utf-8"
       })
.then(function (response) {
      var dataToReturn = [];
      
      for(var i = 0; i < response.length; i++) {
      var tagToTransform = response[i];
      var newTag = {
      id: tagToTransform["name"],
      text: tagToTransform["name"]
      };
      
      dataToReturn.push(newTag);
      }
      
      $("#categories").select2({
                               placeholder: "Select Categories for the Acronym",
                               tags: true,
                               tokenSeparator: [','],
                               data: dataToReturn
                               });
      });
```

![select2 category tags](/assets/select2-category-tags.png)

*acronym.leaf*
```html
#set("content"){
...
    #if(count(categories) > 0) {
        <h3>Categories</h3>
        <ul>
            #for(category in categories) {
            <li>
                <a href="/category/#(category.id)">
                    #(category.name)
                </a>
            </li>
            }
        </ul>
    }
    <form method="post" action="/acronyms/#(acronym.id)/delete">
        <a class="btn btn-primary" href="/acronyms/#(acronym.id)/edit" role="button">Edit</a>&nbsp;
        <input class="btn btn-danger" type="submit" value="Delete" />
    </form>
}

#embed("base")

```

*WebsiteController.swift*
```swift
...
struct AcronymContext: Encodable {
    let title: String
    let acronym: Acronym
    let user: User
    let categories: Future<[Category]>
}
...
func acronymHandler(_ req: Request) throws -> Future<View> {
    return try req.parameters.next(Acronym.self)
        .flatMap(to: View.self) { acronym in
            return acronym.user.get(on: req)
                .flatMap(to: View.self) { user in
                    let categories = try acronym.categories.query(on: req).all()
                    let context = AcronymContext(
                        title: acronym.short,
                        acronym: acronym,
                        user: user,
                        categories: categories)
                    return try req.view().render("acronym", context)
            }
    }
}
...
```

![acronym categories](/assets/acronym-categories.png)