!!! tip "REST"
    - REST是 Representational State Transfer 的缩写，是一种分布式超媒体系统的架构风格

    - 像其它架构风格一样，它有自己的规则和规范。如果一个API服务满足REST架构规范，就可以被称为 **RESTful API**
    
    - 通过一个[RESTful API教程][rest api tutorial]，具体了解REST架构

遵循REST风格的API可以和数据库操作CRUD(创建、查询、更新、删除)联系起来。REST风格的API可以用一种统一的模式来操作资源，这样可以简化客户端的构建过程。

例如要开发一套首字母缩写相关的API，遵循REST风格，可以这样定义：

!!! info "RESTful API定义示例"
    **创建**

    - `POST /api/acronuyms/1`，创建一个ID为1的首字母缩写

    **查询**

    - `GET /api/acronyms/`，获取全部的首字母缩写
    - `GET /api/acronyms/1`，获取ID为1的首字母缩写

    **更新**

    - `PUT /api/acronyms/1`，用新的内容更新ID为1的首字母缩写

    **删除**

    - `DELETE /api/acronyms/1`，删除ID为1的首字线缩写

通过API的接口定义就可以清楚的了解API的具体功能作用，接口定义也比较规范和整洁，一般不需要注释就可以理解
  
[rest api tutorial]: <https://restfulapi.net/>