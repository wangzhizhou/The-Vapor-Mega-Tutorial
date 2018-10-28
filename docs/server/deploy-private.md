Vapor Cloud是很方便部署的，但是随时可能会变成收费的。假和你自己已经投资入手了云服务器，完成可以把项目放在自己的服务器上运行。

首先在ubuntu上安装Docker，之后用docker安装数据库容器，原理与前述一致，因为数据库只在服务器上本地使用，所有不需要暴露给外面使用

```bash
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce
$ curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s  http://f1361db2.m.daocloud.io
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
$ docker run --name postgres \
-e POSTGRES_DB=vapor \
-e POSTGRES_USER=vapor \
-e POSTGRES_PASSWORD=password \
-p 5432:5432 \
-d postgres
$ docker run --name postgres-test \
-e POSTGRES_DB=vapor-test \
-e POSTGRES_USER=vapor \
-e POSTGRES_PASSWORD=password \
-p 5433:5432 \
-d postgres
$ vapor run --hostname=0.0.0.0 --port=8080
```
云服务器需要开端口权限，域名需要配置解析，使用nginx进行端口转发


最终完成API的项目地址: <https://github.com/wangzhizhou/TILApp-BackEnd/releases/tag/1.0>