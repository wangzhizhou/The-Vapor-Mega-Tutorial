!!! tip "使用Xcode开发时，如果找不到文件或者项目配置出错时尝试重新生成Xcode项目"
    ```bash 
    vapor clean --swiftpm && vapor xcode 
    ```

!!! hint "完全找不到错误原因时尝试全部清理重新编译运行"
    1. Xcode中使用Command+Option+K 清空编译产物

    2. 清空编译中间产物
    ```bash 
    vapor clean
    ```

    3. 清空所有产物
    ```bash
    vapor clean --update --global --swiftpm
    ```
    或者
    ```bash
    vapor clean -u -g -s
    ```

!!! hint "求助他人"
    Vapor在[Discord Channel][discord channel]上有频道可以求助。但Discord在中国完全被墙掉了，这也对于Vapor的流行起到了一定的阻碍作用。如果有访问国际互联网的手段，翻墙后可以下载[`Discord`][discord download]，并进入频道

!!! warning "TODO: 为了更好的进行Vapor学习交流，也需要建立一个讨论群"
    

[discord channel]: <https://discord.gg/vapor>
[discord download]: <https://discord.onl/download/>