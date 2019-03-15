# 顶级目录下的文件

```bash
$ tree . -L 1
.
├── CONTRIBUTING.md    # api模板工程参与改进的方式说明，你可以参与改进这个模板工程，如果有想法的话，可以添加一些新特性，但需要按照这个文件里面的说明流程去做
├── Package.resolved # 这是模板工程所依赖的其它包的解析结果，包含了一些版本信息、包名称和仓库地址，工程在编译时使用的包就是这个文件中指定的，它是在解析Package.swift时自动生成的，开发者不需要修改
├── Package.swift # 这个是工程开发者指定项目依赖包版本和仓库地址信息的文件，这个文件是工程开发者手工输入指定的依赖关系文件。用来定义一个项目的依赖和可以生成的产物信息
├── Public # 这个目录用来存放工程需要用到和一些公共资源，例如图片、音频、文本、样式表、js等
├── README.md # 这个是模板工程的说明文件，是工程项目自己的介绍文件，用来展示工程的使用方法和一些其它说明，是这个工程的代表性文件。
├── Sources # 用来存放整个工程的主体源代码文件
├── Tests # 用来存放针对工程功能所写的测试用例的代码文件。
├── cloud.yml # 用来存放工程部署在Vapor Cloud上时所需要的一些配置信息，例如允许访问的主机IP范围和服务监听的端口号,以后需要用到的Swift运行环境的版本
└── web.Dockerfile # 针应用部署到docker时的执行配置文件，方便本地部署。

3 directories, 6 files
```

## Source目录

```bash
$ tree -L 3 Sources/
Sources/
├── App
│   ├── Controllers
│   │   └── TodoController.swift
│   ├── Models
│   │   └── Todo.swift
│   ├── app.swift
│   ├── boot.swift
│   ├── configure.swift
│   └── routes.swift
└── Run
    └── main.swift

4 directories, 7 files
```

Source目录下的每一个子目录都是项目的一个模块。`App`是这个工程的应用核心模块，Run是基于这个App模块，用来启动App模块的另一个模块。整个工程运行的入口是`main.swift`文件。`Package.swift`文件中指定了它们之间的相互关系：

```swift
$ cat Package.swift 
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "HellVapor",
    dependencies: [
        // 💧 A server-side Swift web framework.
        .package(url: "https://github.com/vapor/vapor.git", from: "3.0.0"),

        // 🔵 Swift ORM (queries, models, relations, etc) built on SQLite 3.
        .package(url: "https://github.com/vapor/fluent-sqlite.git", from: "3.0.0")
    ],
    targets: [
        .target(name: "App", dependencies: ["FluentSQLite", "Vapor"]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"])
    ]
)
```

从代码中可以看出，`App`模块依赖了两个包: `Vapor`和`FluentSQLite`，这两个被依赖的包的信息在上面的`dependencies`数组中指定，`SPM`会解析它，并拉取相关的文件到本地参与工程编译。

`Run`模块依赖了`App`模块，它在更上的一层，在本工程中就是用来启动和运行App模块的，为App模块的运行提供了一个入口。

## Tests目录

`AppTests`模块依赖了`App`模块，因为它是针对`App`专门写的测试模块，通过运行一个个测试用例，来测试`App`模块的各个功能是否正常。

`Tests`目录下面有一个`LinuxMain.swift`文件，这个是因为在MacOS和Linux上跑测试用例的实现方式有些差异，需要分别进行相关的配置，所以`LinuxMain.swift`对应Linux平台上进行测试时的配置。

**其实源码内容不太多，可以好好的看一遍来理解整个运行过程，这对之后的学习有很大的帮助**

// TODO: 整理一个工程运行流程图来说明整个过程。