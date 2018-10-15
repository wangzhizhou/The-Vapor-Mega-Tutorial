!!! hint "找不到文件或者项目配置出错时尝试重新生成XCode项目"
    ```bash 
    $ vapor xcode -y 
    ```



!!! hint "依赖库有问题导致bug时尝试更新依赖库"
    删除Package.resolved文件，执行更新 
    ```bash
    $ vapor update
    ```

!!! hint "完全找不到错误原因时尝试全部清理重新编译运行"
    1. Command+Option+K 清空编译产物

    2. 清空编译中间产物
    ```bash 
    $ rm -rf .build
    ```

!!! hint "求助他人"
    Vapor在Discord上有频道可以求助