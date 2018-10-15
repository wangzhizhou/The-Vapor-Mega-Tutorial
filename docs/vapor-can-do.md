如果你上一节中使用了`vapor --help`命令，你会发现一些有用的信息，其中就有告诉你Vapor能作哪些事情。

```bash
$ vapor --help
Usage: vapor command

Join our team chat if you have questions, need help,
or want to contribute: http://vapor.team

Commands:
       new Creates a new Vapor application from a template.
           Use --template=repo/template for github templates
           Use --template=full-url-here.git for non github templates
           Use --web to create a new web app
           Use --auth to create a new authenticated API app
           Use --api (default) to create a new API
     build Compiles the application.
       run Runs the compiled application.
     fetch Fetches the application's dependencies.
    update Updates your dependencies.
     clean Cleans temporary files--usually fixes
           a plethora of bizarre build errors.
      test Runs the application's tests.
     xcode Generates an Xcode project for development.
           Additionally links commonly used libraries.
   version Displays Vapor CLI version
     cloud Commands for interacting with Vapor Cloud.
    heroku Commands to help deploy to Heroku.
  provider Commands to help manage providers.

Use `vapor command --help` for more information on a command.
```

`Vapor`可以通过`new`命令，从模板创建工程，这种工程模板可以是自己定义的，也可以是Vapor默认提供的。自定义的工程模板可以放在本地，也可以放在github上。默认提供的模板有：`Web模板`、`身份认证模板`、`后端API开发模板`。其它的一些命令会在之后的实践中慢慢用到，我们可以先不用理会。

!!! Example "使用API模板创建项目"
    ```bash
    $ mkdir ~/vapor
    $ cd ~/vapor
    $ vapor new HelloVapor
    Cloning Template [Done]
    Updating Package Name [Done]
    Initializing git repository [Done]     

                                           **
                                         **~~**
                                       **~~~~~~**
                                     **~~~~~~~~~~**
                                   **~~~~~~~~~~~~~~**
                                 **~~~~~~~~~~~~~~~~~~**
                               **~~~~~~~~~~~~~~~~~~~~~~**
                              **~~~~~~~~~~~~~~~~~~~~~~~~**
                             **~~~~~~~~~~~~~~~~~~~~~~~~~~**
                            **~~~~~~~~~~~~~~~~~~~~~~~~~~~~**
                            **~~~~~~~~~~~~~~~~~~~~~~~~~~~~**
                            **~~~~~~~~~~~~~~~~~~~~~++++~~~**
                             **~~~~~~~~~~~~~~~~~~~++++~~~**
                              ***~~~~~~~~~~~~~~~++++~~~***
                                ****~~~~~~~~~~++++~~****
                                   *****~~~~~~~~~*****
                                      *************
                             
                             _       __    ___   ___   ___
                            \ \  /  / /\  | |_) / / \ | |_)
                             \_\/  /_/--\ |_|   \_\_/ |_| \
                               a web framework for Swift     

                         Project "HelloVapor" has been created.
                  Type `cd HelloVapor` to enter the project directory.
                Use `vapor cloud deploy` to host your project for free!
                                         Enjoy!    
    

    $ cd HelloVapor
    $ vapor build
    Building Project [Done]
    $ vapor run
    Running HelloVapor ...
    [ INFO ] Migrating 'sqlite' database    (/Users/joker/Desktop/vapor/HelloVapor/.build/checkouts/fluent.   git-8483266616959926579/Sources/Fluent/Migration/MigrationConfig.  swift:69)
    [ INFO ] Preparing migration 'Todo'     (/Users/joker/Desktop/vapor/HelloVapor/.build/checkouts/fluent.   git-8483266616959926579/Sources/Fluent/Migration/Migrations.swift:111)
    [ INFO ] Migrations complete    (/Users/joker/Desktop/vapor/HelloVapor/.build/checkouts/fluent.   git-8483266616959926579/Sources/Fluent/Migration/MigrationConfig.  swift:73)
    Running default command: .build/debug/Run serve
    Server starting on http://localhost:8080
    ```

第一次`vapor build`这一步可能会有点慢，因为它要从网络上拉取一些依赖的仓库代码，来构建项目。

项目运行起来默认是在`8080`端口上服务的，而且只能从本机访问，如果要指定端口和允许任意IP地址访问API，可以使用下面的命令运行:
```bash
$ vapor run --hostname=0.0.0.0 --port=80
Running HelloVapor ...
[ INFO ] Migrating 'sqlite' database (/Users/joker/Desktop/vapor/HelloVapor/.build/checkouts/fluent.git-8483266616959926579/Sources/Fluent/Migration/MigrationConfig.swift:69)
[ INFO ] Preparing migration 'Todo' (/Users/joker/Desktop/vapor/HelloVapor/.build/checkouts/fluent.git-8483266616959926579/Sources/Fluent/Migration/Migrations.swift:111)
[ INFO ] Migrations complete (/Users/joker/Desktop/vapor/HelloVapor/.build/checkouts/fluent.git-8483266616959926579/Sources/Fluent/Migration/MigrationConfig.swift:73)
Running default command: .build/debug/Run serve
[Deprecated] --option=value syntax is deprecated. Please use --option value (with no =) instead.
[Deprecated] --option=value syntax is deprecated. Please use --option value (with no =) instead.
Server starting on http://0.0.0.0:80
```

运行起来后，我们访问API服务就可以验证了。模板默认提供一个API服务：`/hello`, 用Chrome访问一下试试：

![chrome it works](/assets/it_works.png)

![chrome hello world](/assets/hello_world.png)
