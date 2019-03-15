
上一节我们通过使用`vapor new`命令，以通用api模板创建了一个项目`HelloVapor`, 那么我们进一步看一下模板是怎么通过目录结构划分应用的功能模块的。

```bash
$ pwd 
/Users/joker/Desktop/vapor/HelloVapor
$ tree -L 3 .
.
├── Package.resolved
├── Package.swift
├── Public
├── README.md
├── Sources
│   ├── App
│   │   ├── Controllers
│   │   ├── Models
│   │   ├── app.swift
│   │   ├── boot.swift
│   │   ├── configure.swift
│   │   └── routes.swift
│   └── Run
│       └── main.swift
├── Tests
│   ├── AppTests
│   │   └── AppTests.swift
│   └── LinuxMain.swift
└── cloud.yml

8 directories, 11 files
```

Vapor使用Swift Package Manager(SPM)，它是一个依赖管理系统，类似于iOS平台上的Cocoapods，用来配置和构建Vapor应用。使用SPM管理的项目可以在.gitignore文件中设置忽略Xcode项目相关的文件，如果想使用Xcode，每次更改都需要重新生成项目文件。

一个SPM项目定义在Package.swift框架文件中，这个文件中声明了目标(Target)、依赖(Dependencies)以及怎样把它们链接在一起。项目的布局也和传统的XCode项目有所不同。

`Source`目录用来存放源文件，每一个在Package.swift中定义的模块都对应`Source` 目录下的一个模块子目录。模板项目中声明了两个模块: `App`和`Run`。

`Run`模块下仅有一个`main.swift`文件，这是swift项目的启动入口。我们在开发过程中不会修改`Run`模块。

`Tests`中存放所有的测试模块，一个测试模块对应一个模块子目录。