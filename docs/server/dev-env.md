`Vapor`的开发环境目前主要是针对`Mac OS`和`Ubuntu`两个平台部署。

!!! info
    在`Mac OS`你要先安装好`Xcode`，这个是为了安装好`Swift`语言环境，Shell里使用`$ swift --version`查看Swift版本


```bash tab="MacOS"
# 安装Homebrew包管理器
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# 使用Homebrew包管理器安装Vapor开发框架
$ brew install vapor/tap/vapor
# 查看框架版本
$ vapor --help
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
# 查看框架版本
$ vapor --help
```

执行完上面的命令，`Vapor`的开发环境就完成了。因为目前来说，Ubuntu上还没有对Swift支持比较好的IDE，所以建议使用Mac上的XCode作为开发工具。

在布置开发环境后可以使用Vapor提供的脚本检查是否安装成功

```bash
$ eval "(curl -sL check.vapor.sh)"
```


!!! warning "端口占用查询"
    在MacOS上，终端键入命令`lsof -i tcp:8080`可以查看指定端口当前被哪些应用使用