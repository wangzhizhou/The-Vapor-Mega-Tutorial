目前，`Vapor`的开发环境主要是在`Mac OS`和`Ubuntu`两个平台部署。

!!! info "MacOS平台上可以借助XCode这个IDE进行开发"
    在`Mac OS`你要先安装好`Xcode`，这个是为了安装好`Swift`语言环境，Shell里使用`$ swift --version`查看Swift版本


下面就是两个平台上部署Vapor开发环境的具体步骤：

```bash tab="MacOS"
# 先从AppStore安装开发工具Xcode(Swift开发环境)
# 安装Homebrew包管理器
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# 使用Homebrew包管理器安装Vapor开发框架
$ brew install vapor/tap/vapor
# 查看vapor工具箱版本
$ vapor version
```

```bash tab="Ubuntu"
# 安装curl工具，用于发起HTTP请求，获取数据
$ sudo apt-get install curl
# 从URL中获取Shell脚本，并在本地Shell中执行
$ eval "$(curl -sL https://apt.vapor.sh)"
# 使用APT包管理工具安装Swift环境和Vapor框架
$ sudo apt-get install swift vapor
# 查看Swift版本
$ swift --version
# 查看vapor工具箱版本
$ vapor version
```

执行完上面的命令，`Vapor`的开发环境就完成了。因为目前来说，Ubuntu上还没有对Swift支持比较好的IDE，所以建议使用Mac上的XCode作为开发工具。

在布置开发环境后可以使用Vapor提供的脚本检查是否安装成功

```bash tab="MacOS"
$ eval "$(curl -sL check.vapor.sh)"
✅  Xcode 10 is compatible with Vapor 2.
✅  Xcode 10 is compatible with Vapor 3.

✅  Swift 4.2 is compatible with Vapor 2.
✅  Swift 4.2 is compatible with Vapor 3.
```

```bash tab="Ubuntu"
$ eval "$(curl -sL check.vapor.sh)"
✅  Swift 4.2 is compatible with Vapor 2.
✅  Swift 4.2 is compatible with Vapor 3.
```

做到这里，Swift开发环境和Vapor工具箱已经安装成功，下一步就可以尝试开发了。🙃🙃🙃