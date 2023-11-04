# CentOS 7.6 通过 docker 安装 mongoDB 7.0.2

> 参考链接：[Docker-从入门到实践](https://yeasy.gitbook.io/docker_practice/) , [MongoDB官网](https://www.mongodb.com/zh-cn)

最近一个项目中需要用到 mongoDB，俗话说的好万事开头难，光安装配置数据库就耗费了整整一个晚上，赶紧记下来不然回头又忘了

## 1.安装 docker

我的服务器是 Centos 7.6，网上安装教程一大把，这里就不再赘述了，其实不需要那么多第三方教程，直接上官网一步一步来就可以了

 [docker 官网](https://www.docker.com/)

## 2. 在 docker 中安装 mongoDB

```shell
# 先拉取 mongoDB 镜像
docker pull mongo:latest
# 查看镜像是否下载成功
docker images
```

**重点来了！！！** 让我想启动带访问控制（需要用户名密码才能连接）的 mongoDB 容器时，发现了个很离谱的问题，苦难与折磨在等着我呢

```shell
# 错误演示错误演示错误演示！！！！！不要这么用！！！！

# 正常启动一个带访问控制的 mongo 容器
docker run -d --name mongo mongo --auth
# 进入容器增加用户
docker exec -it mongo mongosh admin
# 创建用户
admin> db.createUser({ user:'root',pwd:'root',roles:[ { role:'root', db: 'admin'}]});
# 然后会让我去验证管理员账号密码,这个时候我压根就没有管理员用户(黑人问号？？？)
admin> db.auth("root","root")

# 然后我删掉了这个容器
docker container rm -f mongo
# 重新创建了一个不需要访问控制的容器
docker run -d --name mongo mongo
# 进入容器增加用户
# 创建用户
admin> db.createUser({ user:'root',pwd:'root',roles:[ { role:'root', db: 'admin'}]});
# 验证一下
admin> db.auth("root","root")
# 验证成功
{ok : 1}
# 在容器中安装 vim 然后修改 mongo 配置文件
apt-get update
apt-get install vim
vim /etc/mongod.conf.orig

security:
    authorization：enabled # 这里设置成 enabled

:wq

# 然后我发现重启 mongo 服务和 mongo 容器都没反应？？？
```

我看了很多资料，说把 mongo 容器备份复制，设置环境变量，换老版本什么什么的，老大我就是一个 docker + mongoDB 新手，人菜脾气大，就想用最新版本的，我相信很多同学装 mongoDB 也是刚入门学习的，不应该简单点，先装好，然后会使用，最后懂原理才对嘛

在无尽的痛苦和煎熬中我发现一个可以指定 mongo 数据储存位置的参数，于是我就在想

1. 先创建一个**不带**访问控制的 mongoDB 容器，然后创建一个管理员
2. 再创建一个**带**访问控制的 mongoDB 容器，数据储存位置跟上一个相同
3. 删掉第一个容器，这不就得了

```shell
# 先创建一个不带访问控制的容器, 用 -v 指定储存位置
docker run -d --name mongo_unsafe -v ~/docker/mongo:/data/db mongo
# 进入容器并创建用户
docker exec -it mongo_unsafe mongosh admin
admin> db.createUser({ user:'root',pwd:'root',roles:[ { role:'root', db: 'admin'}]});
# 验证一下
admin> db.auth("root","root")
# 验证成功
{ok : 1}
#  退出并停止这个容器，因为创建新的 mongo 容器默认端口也是 27017
docker stop mongo_unsafe
# 创建带访问控制的容器，为了减少后续的折磨，这回加上自动重启和端口映射，提高安全性
# -v 后面的路径跟上面是一致的
# -p 前面的数字是 mongo 运行端口，默认是27017，后面的是你想通过外部访问的端口，别写错了
docker run -d --restart=always -p 27017: 外部访问端口 --name mongo -v ~/docker/mongo:/data/db mongo
# 直接进入容器并验证
docker exec -it mongo mongosh admin
admin> db.auth("root","root")
# 验证成功
{ok : 1}
# 退出然后删掉第一个容器
docker container rm -f mongo_unsafe
```

然后用 mongo compass 之类的 GUI 测试一下连接正常，苦难与折磨烟消云散了









