目前，`Vapor`的开发环境主要是在`Mac OS`和`Ubuntu`两个平台部署。

!!! info "MacOS平台上可以借助XCode这个IDE进行开发"
    在`Mac OS`你要先安装好`Xcode`，这个是为了安装好`Swift`语言环境，Shell里使用`$ swift --version`查看Swift版本, 要求Swift版本在`5.2`及以上。


下面是`MacOS`和`Ubuntu`两个系统平台上部署Vapor开发环境的具体步骤：

=== "MacOS"
    1. 先从`App Store`安装开发工具[Xcode](https://apps.apple.com/cn/app/xcode/id497799835?mt=12)(Swift开发环境)
    2. 再到[Homebrew官网](https://homebrew.sh)，安装Homebrew包管理器
    ```bash
    $ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```
    3. 使用Homebrew包管理器安装Vapor开发框架
    ``` bash
    $ brew install vapor/tap/vapor
    ```
=== "Ubuntu"
    1.浏览[`Swift`官网](https://swift.org/download)，下载Swift工具链并安装开发环境，安装完成后查看swift版本号
    ``` bash 
    $ swift --version
    ```
    2.从源码安装Vapor
    ```bash 
    $ git clone https://github.com/vapor/toolbox.git
    $ cd toolbox
    $ git checkout 18.0.0
    $ swift build -c release --disable-sandbox
    $ mv ./build/release/vapor /usr/local/bin
    ```

--- 

执行完上面的步骤，`Vapor`的开发环境就布置完成了。因为目前来说，Ubuntu上还没有对Swift支持比较好的IDE，所以建议使用Mac上的XCode作为开发工具。Ubuntu图形界面可以使用[`Visual Studio Code`](https://code.visualstudio.com)，命令行下也可以使用[`Vim`](https://github.com/vim/vim)编辑器。


Swift开发环境和Vapor工具箱安装成功后，下一步就可以尝试开发了。🙃🙃🙃