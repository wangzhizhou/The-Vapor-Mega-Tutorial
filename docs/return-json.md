
Content协议也可以编码结构体成为JSON数据，在代码中定义一个遵循Content协议的结构体InfoResponse，使用请求数据初始化一个响应结构体对象，直接返回，JSON编码会自动完成，并返回JSON数据给用户。

![return json code](/assets/return-json-code.png)


使用`resetd`应用测试如下：

![return json](/assets/return-json.png)