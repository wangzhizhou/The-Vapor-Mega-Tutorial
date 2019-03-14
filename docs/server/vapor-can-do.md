如果你上一节中使用了`vapor --help`命令，你会发现一些有用的信息，其中就有告诉你Vapor工具箱能做哪些事情。

Vapor工具箱是一个命令行工具，可以帮助开发Vapor应用。它可以从指定模板创建应用、可以调用Swift工具链来构建和运行项目、
可以生成项目的对应Xcode版本工程文件，除此之外，还可以把应用部署到Vapor云上进行运行和管理。


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

`Vapor`可以通过`new`命令，从模板创建工程，这种工程模板可以是自己定义的，也可以是Vapor默认提供的。自定义的工程模板可以放在本地，
也可以放在github上。官方提供的模板有：`Web应用模板`、`身份验证API模板`、`通用API开发模板(未指定时，默认生成这种类型)`。
其它的一些命令会在之后的实践中慢慢用到，我们会慢慢的熟悉并熟练的使用它，不必着急。

`vapor build`用来构建应用，`vapor run`运行构建完成的应用。在构建之前，vapor需要按工程依赖关系拉取指定依赖，`vapor fetch`就是完成这个工作。
但工程的依赖由其它开发者维护，所以可能会升级，如果我们之前拉取的依赖是升级前的版本，此时我们可以使用`vapor update`来拉取依赖的最新版本。
`vapor clean`是用来清理构建过程中生成的一些临时文件或中间产物，这些东西有时会引起一些奇怪的构建错误，所以在发生莫名的构建错误时，可以尝试使用这个命令清理一下临时文件，
重新构建，看看能不能解决问题。`vapor test`用来运行应用的测试用例，例如单元测试用例，需要开发者额外编写，以保证应用迭代的质量。
`vapor xcode`用来为应用生成对应能在Xcode中打开的工程文件，方便使用IDE进行开发，这个只针对MacOS平台，因为Ubuntu上没有Xcode。
`vapor version`用来查看vapor工具箱的版本信息。`vapor cloud`用来将开发的应用部署到vapor云服务器上，它包含一些子命令，可以使用`vapor cloud --help`获取更多信息。
`vapor heroku`是把应用部署到heroko云平台上，这个平台也是很有名气的。最后`vapor provider`用来管理供应商，这个用的少，还没有试过。


下面使用`vapor new`来创建一个项目，作为示例。

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

!!! question 
    依赖是通过`Swift Package Manager(SPM)`来获取的。这里有个疑问，就是`SPM`并不像`cocoapods`
    这类工具能够统一修改依赖的源地址，在`SPM`中的依赖地址都是写死的，这不利于代码迁移到内部服务器上使用。有没有知道解决方案的，可以联系微信: `w_z_z_1991`告知我。

    目前从模板新建一个项目以及拉取依赖的耗时比较长，所以得想办法解决这个问题。否则，对于vapor的推广很不利。我在自己的阿里云服务器上(双核4G内存共享计算式)使用模板创建一个项目的耗时如下：

    ```bash
    $ time vapor new HelloVapor
    ...
    real    2m11.103s
    user    0m0.591s
    sys     0m0.365s
    ```
    可见，创建一个项目居然要耗时2分钟左右的时间，这是无法忍受的。再来看一下解析依赖的时间：

    ```bash
    $ cd HelloVapor && time vapor fetch
    Fetching Dependencies [Done]

    real    18m18.362s
    user    0m15.915s
    sys     0m6.833s
    ```
    第一次拉取依赖耗时18分钟左右，因为是要在Github上的仓库上拉取。
    
    如果想要以swift+vapor的方式创业，那么迁移依赖库到私人服务器势在必行。好在个人开发来说，创建项目和依赖解析都只会在开始的时候最耗时，暂时可以接受。

    **TODO**: 调研这个问题的解析方案。

    初始构建耗时较大，之后就是增量构建：
    ```bash
    $ time vapor build
    Building Project [Done]

    real    0m52.346s
    user    1m26.204s
    sys     0m5.493s
    ```

当对`HelloVapor`工程完成前面的`vapor build`这一步以后，就可以查看这个api项目提供的所有可用路由信息了： 

```bash
$ vapor run routes
Running HellVapor ...
[ INFO ] Migrating 'sqlite' database (/Users/joker/Documents/HellVapor/.build/checkouts/fluent.git-1559603717901448874/Sources/Fluent/Migration/MigrationConfig.swift:69)
[ INFO ] Preparing migration 'Todo' (/Users/joker/Documents/HellVapor/.build/checkouts/fluent.git-1559603717901448874/Sources/Fluent/Migration/Migrations.swift:111)
[ INFO ] Migrations complete (/Users/joker/Documents/HellVapor/.build/checkouts/fluent.git-1559603717901448874/Sources/Fluent/Migration/MigrationConfig.swift:73)
+--------+--------------+
| GET    |              |
+--------+--------------+
| GET    | /hello       |
+--------+--------------+
| GET    | /todos       |
+--------+--------------+
| POST   | /todos       |
+--------+--------------+
| DELETE | /todos/:todo |
+--------+--------------+
```

使用`vapor run`命令运行项目，默认是在监听`127.0.0.1:8080`，所以你只能从本机访问。如果要指定端口和允许任意IP地址访问API，可以使用下面的命令运行:
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

运行起来后，我们就可以在内网中的任何设备上访问了到刚刚开发的API了，例如可以使用手机浏览器访问电脑的ip，验证API服务正常工作。模板默认提供一个API服务：`/hello`, 用Chrome(本机上打开的)访问一下试试：

![chrome it works](/assets/it_works.png)

![chrome hello world](/assets/hello_world.png)


!!! warning "端口占用查询"
    有时在运行应用时会发现想要监听的端口已经被其它的应用程序绑定了，此时可以使用命令查看一下到底是谁在使用。
    在MacOS上，终端键入命令`lsof -i tcp:8080`可以查看指定端口当前被哪些应用使用。然后使用`pkill APP_NAME`来关闭这些占用端口的应用。
    如果占用端口的应用很重要，那么你就要考虑换一个端口来监听运行你的API了。