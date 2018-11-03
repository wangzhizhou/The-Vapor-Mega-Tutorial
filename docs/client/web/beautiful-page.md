#模板嵌套

Leaf模板可以相互嵌套，达到html代码复用的目的。

首先把`index.leaf`中的复用的部分抽出到`base.leaf`，以供其它页面复用。

*base.leaf*
```html
<!DOCTYPE html>

<html lang="en">
    <head>
        <meta charset="utf-8" />
        <title>#(title) | Acronyms</title>
    </head>
    <body>
        #get(content)
    </body>
</html>
```

*index.leaf*
```html
#set("content") {
    <h1>Acronyms</h1>
    #if(acronyms) {
    <table>
        <thead>
            <tr>
                <th>Short</th>
                <th>Long</th>
            </tr>
        </thead>
        <tbody>
            #for(acronym in acronyms) {
            <tr>
                <td><a href="/acronyms/#(acronym.id)">#(acronym.short)</a></td>
                <td>#(acronym.long)</td>
                }
                </tbody>
    </table>
    } else {
    <h2>There aren't any acronyms yet!</h2>
    }
}

#embed("base")
``` 

使用了Leaf中的tag: #get()、#set()、#embed()。其中，#set()需要用引号使Leaf注册变量。base.leaf就是一种模板可以被index.leaf复用。

# Bootstarp

Bootstrap是一个开源、前端网站框架，由Twitter建立，提供很多易用的组件。它同时是移动端优先支持的，可以适用于多种尽寸的屏幕大小。它提供了一个css文件定义各种显示样式，还有一些js文件提供各种组件。我们需要把这些文件包含在所有的页面中使用，为了避免代码重复编写，把它们加在模板文件`base.leaf`中在其它页面中复用就不需要每个页面包含一次这些文件的繁琐操作了。

官网提供了一个使用Bootstrap的模板，我们照着改造一下我们自己的模板文件`base.leaf`：

![bootstrap-start-template](/assets/bootstrap-starter-template.png)


```html
<!DOCTYPE html>

<html lang="en">
    <head>
        <!-- Required meta tags -->
        <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
                
                <!-- Bootstrap CSS -->
                <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
                    
                    <title>#(title) | Acronyms</title>
                    </head>
    <body>
        <nav class="navbar navbar-expand-md navbar-dark bg-dark">
            <a class="navbar-brand" href = "/">TIL</a>
            <button class="navbar-toggler" type="button"
                data-toggle="collapse" data-target="#navbarSupportedContent"
                aria-controls="navbarSupportedContent" aria-expanded="false"
                aria-label="Toggle navigation">>
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id = "navbarSupportedContent">
                <ul class="navbar-nav mr-auto">
                    <li class="nav-item #if(title == "Homepage"){active}">
                        <a href="/" class="nav-link">Home</a>
                    </li>
                </ul>
            </div>
        </nav>
        <div class="container mt-3">
            #get(content)
        </div>
        
        <!-- Optional JavaScript -->
        <!-- jQuery first, then Popper.js, then Bootstrap JS -->
        <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
    </body>
</html>
```

*index.leaf*
```html
#set("content") {
    <h1>Acronyms</h1>
    #if(acronyms) {
    <table class = "table table-bordered table-hover">
        <thead class="thead-light">
            <tr>
                <th>Short</th>
                <th>Long</th>
            </tr>
        </thead>
        <tbody>
            #for(acronym in acronyms) {
            <tr>
                <td><a href="/acronyms/#(acronym.id)">#(acronym.short)</a></td>
                <td>#(acronym.long)</td>
                }
                </tbody>
    </table>
    } else {
    <h2>There aren't any acronyms yet!</h2>
    }
}

#embed("base")

```


![beautiful page wih bootstrap](/assets/beautiful-page-with-bootstrap.png)