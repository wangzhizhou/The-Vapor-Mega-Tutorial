目前，Vapor 的开发环境主要是在 MacOS 和 Ubuntu 两个平台部署。

!!! info "MacOS平台上可以借助Xcode这个IDE进行开发"
    在`MacOS`你要先安装好`Xcode`，Xcode内置有`Swift`开发环境, `Vapor4`要求Swift版本在`5.2`及以上。
    
    使用shell命令查看Swift版本：
    ```bash
    swift --version
    ```


下面是`MacOS`和`Ubuntu`两个系统平台上部署Vapor开发环境的具体步骤：

=== "MacOS"
    1. 先从`App Store`安装开发工具[Xcode](https://apps.apple.com/cn/app/xcode/id497799835?mt=12)(Swift开发环境)
    2. 再到[Homebrew官网](https://brew.sh/)，安装Homebrew包管理器
    ```bash
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```
    3. 使用Homebrew包管理器安装Vapor开发工具
    ``` bash
    brew install vapor
    ```
=== "Ubuntu"
    1.浏览[`Swift`官网](https://swift.org/download)，按照官方指导，下载 Swift 工具链并手动安装开发环境，安装完成后查看swift版本号：`swift --version`
    ??? info "[自动安装方法] 使用 [Swiftly](https://swift-server.github.io/swiftly/) 命令行工具"
        1. 安装 Swiftly
        ```bash
        $ curl -L https://swift-server.github.io/swiftly/swiftly-install.sh | bash
        ```

        2. 使用Swiftly进行安装最新版 Swift 开发环境
        ```bash
        $ swiftly install latest
        ```
    2.在ubuntu上需要安装 `build-essential`，为Vapor的源码编译做好准备：
    ```bash
    sudo apt-get install -y build-essential libz-dev
    ```
    
    3.从源码安装Vapor
    ```bash 
    git clone https://github.com/vapor/toolbox.git
    cd toolbox
    git checkout <desired version>
    make install
    ```
---

检查Vapor是否安装成功
```bash
vapor --help
```

执行完上面的步骤，`Vapor`的开发环境就布置完成了。


如果你的开发平台是MacOS，建议使用Xcode作为开发工具。目前在Ubuntu上还没有对Swift支持比较好的IDE，可以使用[Visual Studio Code][vsc]安装Swift插件的方式进行开发，命令行下也可以使用[Vim]编辑器进行Swift相关开发。

=== "Xcode(MacOS)"
    只需要在Vapor创建的项目根目录下运行下面的命令，即可打开Xcode工程进行开发
    ```bash
    open Package.swift
    ``` 
    或者
    ```bash
    xed .
    ```
=== "Visual Studio Code(通用)"
    1. 本地需要先安装好Swift环境
    2. 安装 [Visual Studio Code][vsc]
    3. 安装 Visual Studio Code 插件：[sswg.swift-lang][vsc-swift-plugin]，插件的相关文档可以参考：[README](https://github.com/swift-server/vscode-swift#readme)
    4. 使用 Visual Studio Code 打开Vapor工程根目录
    5. 运行、打断点、调试可以直接按常规操作进行
=== "Vim(命令行IDE)"
    !!! warning "TODO: 添加Vim配置Swift开发环境"
---
Swift开发环境和Vapor工具安装成功后，下一步就可以尝试开发了。🙃🙃🙃


[vsc]: <https://code.visualstudio.com>
[vim]: <https://github.com/vim/vim>
[vsc-swift-plugin]: <https://marketplace.visualstudio.com/items?itemName=sswg.swift-lang>