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
    Vapor在[Discord](https://discord.onl/download/)上有频道可以求助。但Discord在中国完成被墙掉了，这也对于Vapor的流行起到了一定的阻碍作用。对于翻墙我这里也提供一个方案，就是使用信用卡申请谷歌云机器用来搭建VPS服务器来播墙上网，由于谷歌云会提供300美元一年的试用金，并且可以重复开通，所以相当划算。具体方法请搜索:`谷歌云+BBR+SSR`。不过现在我翻墙也上不去了，所以还得找其它的爱好者群。或者考虑自己建一个也不错。
