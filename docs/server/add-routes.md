如果你在Linux上开发，可以选择自己喜欢的编辑器来修改文件，然后使用命令行`vapor build`和`vapor run`来编译运行。

如果你在Mac上开发，那么可以使用Xcode这个IDE来进行，首先使用命令行生成Xcode项目并打开工程：

``` bash
$ vapor xcode -y
```
然后添加两个GET类型的服务，如图，在文件routes.swift中添加红框的内容，选择`My Mac`设备, `Run`运行方案，并点击运行按钮

![add routes](/assets/add-routes.png)

使用chrome访问我们添加的路由如下图，可见正常工作，其中第二个路由可以接收变化的参数：

![hello vapor](/assets/hello-vapor.png)

![hello joker](/assets/hello-joker.png)