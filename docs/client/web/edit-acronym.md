*WebsiteController.swift*
```swift
...
struct EditAcronymContext: Encodable {
    let title = "Edit Acronym"
    let acronym: Acronym
    let users: Future<[User]>
    let editing = true
}
...
    router.get("acronyms", Acronym.parameter, "edit", use: editAcronymHandler)
    router.post("acronyms", Acronym.parameter, "edit",  use: editAcronymPostHandler)
...
    func editAcronymHandler(_ req: Request) throws -> Future<View> {
        return try req.parameters.next(Acronym.self)
            .flatMap(to: View.self) { acronym in
                let context = EditAcronymContext(acronym: acronym, users: User.query(on: req).all())
                return try req.view().render("createAcronym", context)
        }
    }
    
    
    func editAcronymPostHandler(_ req: Request) throws -> Future<Response> {
        return try flatMap(to: Response.self,
                           req.parameters.next(Acronym.self),
                           req.content.decode(Acronym.self)
        ) { acronym, data in
            acronym.short = data.short
            acronym.long = data.long
            acronym.userID = data.userID
            
            return acronym.save(on: req).map(to: Response.self) { savedAcronym in
                guard let id = savedAcronym.id else {
                    throw Abort(HTTPResponseStatus.internalServerError)
                }
                return req.redirect(to: "/acronyms/\(id)")
            }
        }
    }
```

*createAcronym.leaf*
```html
#set("content"){
    <h1>#(title)</h1>
    <form method = "post">
        <div class="form-group">
            <label for = "short">Acronym</label>
            <input type="text" name="short" class="form-control" id = "short" #if(editing) { value = "#(acronym.short)" }/>
        </div>
        <div class="form-group">
            <label for = "long">Meaning</label>
            <input type="text" name="long" class="form-control" id="long" #if(editing) { value = "#(acronym.long)"}/>
        </div>
        <div class="form-group">
            <label for="userID">User</label>
            <select name="userID" class="form-control" id="userID">
                #for(user in users) {
                    <option value="#(user.id)" #if(editing){ #if(acronym.userID == user.id) { selected }}>
                        #(user.name)
                    </option>
                }
            </select>
        </div>
        <button type="submit" class = "btn btn-primary">
            #if(editing) { Update } else { Submit }
        </button>
    </form>
}

#embed("base")
```

*acronym.leaf*
```html
#set("content"){
    <h1>#(acronym.short)</h1>
    <h2>#(acronym.long)</h2>
    <p>Created by <a href="/users/#(user.id)/">#(user.name)</a></p>
    <a class="btn btn-primary" href="/acronyms/#(acronym.id)/edit" role="button">Edit</a>
}

#embed("base")
```

![edit acronym](/assets/edit-acronym.png)

![update acronym](/assets/update-acronym.png)