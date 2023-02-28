!!! tip "使用Xcode开发时，如果找不到文件或者项目配置出错时尝试重新生成Xcode项目"
    ```bash 
    rm -rf .swiftpm
    vapor xcode 
    ```



!!! hint "依赖库有问题导致bug时尝试更新依赖库"
    执行依赖更新命令 
    ```bash
    swift package update
    ```

!!! hint "完全找不到错误原因时尝试全部清理重新编译运行"
    1. Xcode中使用Command+Option+K 清空编译产物

    2. 清空编译中间产物
    ```bash 
    rm -rf .build
    ```

!!! hint "求助他人"
    Vapor在[Discord Channel][discord channel]上有频道可以求助。但Discord在中国完全被墙掉了，这也对于Vapor的流行起到了一定的阻碍作用。如果有访问国际互联网的手段，翻墙后可以下载[`Discord`][discord download]，并进入频道

[discord channel]: <https://discord.gg/vapor>
[discord download]: <https://discord.onl/download/>