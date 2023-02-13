# 写完代码以后的事-Nginx 单域名+单证书配置多项目

*作为一个 .net 入门的软件工程专业毕业生，平时主要用 Windows Server服务器，所以 Linux 运维知识很是欠缺，面对 Linux 繁多的命令一直无从下手，导致恶性循环，直到[宝塔面板](https://www.bt.cn/new/index.html)出现了*

## 背景起因

我们公司信息部门比较小，但是分工还算比较明确。我只需负责编码工作，服务器采购一系列流程都有运维大哥帮忙处理。

前两天搞定一个项目，主要由 微信小程序 + react 管理后台 + nestjs 服务端 组成，前两者共用一个服务端，由于小程序要求服务端必须配置域名 + https 加密访问，所以只能拜托运维大哥给安排一下，可大哥只给了一个域名 + https。

可能因为个人项目比较小，我的习惯是一个端口绑定一个子域名，从来没 域名 + 端口号这么搞过，这次记录一下初体验吧～

## Linux 服务器神器-宝塔面板

> 下载地址： https://www.bt.cn/new/download.html

其实这个面板不止有 Linux （包括 CentOS 和 Ubuntu 等主流发行版），还支持 Windows Server （比如为了省硬盘空间提高效率而不装图形界面，使用 PowerShell 甚至 cmd 与之交互的地狱难度运维），还有手机App等周边服务和增值项目，免费版就足够用了，省去很多手敲命令的麻烦，对我这种 Linux 菜鸡极其友好

安装配置过程不做赘述，下面直接配置环境

![首页](https://static.yuehaowei.fun/static/blog-images/Linux/2.png)

由于需要部署一个 nestjs 服务端 + 一个 react 管理后台，所以我们只需要用到 nginx + pm2。点击左面的软件商店，分别输入 nginx 和 pm2 即可，其中 pm2 自带 node、nvm 等插件，省去手动安装的烦恼

![软件商店](https://static.yuehaowei.fun/static/blog-images/Linux/3.png)

## PM2 部署 nodejs （nestjs）项目

在开始配置之前，我们需要先把 nodejs 项目上传到服务器上

![上传目录](https://static.yuehaowei.fun/static/blog-images/Linux/4.png)

- 点击左侧 **文件**
- 点击上方 **上传** 按钮
- 点击弹框上的 **上传文件** 按钮

当然 nodejs 项目需要依赖项目中的 *node_modules* ，所以我们需要上传整个项目，如果你的项目体积比较大或者服务器带宽限制，**强烈建议将项目整体压缩然后上传一个压缩包**，这样效率会高很多

这时候就可以开始配置 pm2 了

![配置pm2](https://static.yuehaowei.fun/static/blog-images/Linux/5.png)

1. 启动文件就是你 nodejs 项目的启动文件，比如 ```express.js``` 就是 ``` www``` ,像 ```nest.js``` 就是 ```main.js``` ，不过要注意的是如果你的项目跟我一样用了 ```typescript``` ,那得选择编译后 output 文件夹中的 js 文件
2. 运行目录就是上方启动文件所在目录，使用 ```typescript``` 注意事项同上
3. 项目名称随意填写，下方默认就可以了，点击提交

![pm2首页](https://static.yuehaowei.fun/static/blog-images/Linux/6.png)

如果一切正常，你的 pm2 管理器的项目列表会出现刚才你配置的项目，注意端口号是系统自动读取你项目配置文件的端口号，如果你不配置 nginx 反向代理直接 IP + 端口访问的话，需要注意一下端口是否开放或者是否冲突

## Nginx 配置前端项目

当使用 pm2 部署好你的服务端以后，不要忘了先在本地开发环境把接口地址改一下，然后再把前端项目上传到服务器上，方法跟上面上传服务端项目相同

**请注意！！！** 这时候在前端修改的服务端地址应该就是 域名+端口号的，改完以后前端页面会找不到接口地址，先不要着急，一步一步来。

![网站](https://static.yuehaowei.fun/static/blog-images/Linux/7.png)

当你下载安装完成 nginx 以后，点击左侧网站，右侧可以看到上方有项目的分类，前端网站选择 PHP项目即可，然后点击 **添加站点** 按钮

![添加站点](https://static.yuehaowei.fun/static/blog-images/Linux/8.png)

添加站点其实只需要配置两块内容：

1. 域名-这个就靠从公司运维大哥那刷脸刷过来的，我的习惯是把80或者443端口给前端项目用，这样用户不会看到具体的端口名，也不会感到很奇怪，其他的端口用来跑服务端
2. 根目录-就是前面上传前端项目的目录，这个不做赘述

这时候直接点击提交即可～然后打开浏览器，把刚才配置的域名输入到浏览器中，这时候你应该可以看到项目加载出来但是功能并不能用，因为我们还没有配置服务端的反向代理

## 使用 Nginx 反向代理服务端端实现 域名 + 端口号 访问接口

![](https://static.yuehaowei.fun/static/blog-images/Linux/9.png)

返回网站页面，点击 **设置** 按钮

![](https://static.yuehaowei.fun/static/blog-images/Linux/10.png)

点击左侧配置文件按钮，这个时候你的配置文件大概应该是这个样子的，如果你没有配置证书的话应该就是 80 端口这都不影响

因为宝塔面板现在应该还不支持增加一个域名多个端口访问的网站，我试了好多次都没有实现，期待以后可以直接配置～

只能修改配置文件如下

```nginx
server{
  	listen 10001 ssl http2; # 10001 是自定义的端口号，需要根据自己情况填写
    server_name xxx.yyy.com; # 这里填写你的域名
    index index.php index.html index.htm default.php default.htm default.html;
    #SSL-START SSL相关配置，请勿删除或修改下一行带注释的404规则
    #error_page 404/404.html;
    ssl_certificate    /www/server/panel/vhost/cert/xxx.yyy.com/fullchain.pem;
    ssl_certificate_key    /www/server/panel/vhost/cert/xxx.yyy.com/privkey.pem;
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    add_header Strict-Transport-Security "max-age=31536000";
    error_page 497  https://$host$request_uri;
		#SSL-END
  
  	# 配置反向代理
    location / {
    proxy_pass  http://127.0.0.1:8080; # 这里 8080 端口是 pm2 识别服务端项目配置的端口号，这里你也需要自己配置
    proxy_set_header Host $proxy_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
server{
	# 上方自动生成的配置
}
```

这样就配置成功了，其实如果没有配置证书的话上面的 SSL 配置就不存在～

这个时候再打开浏览器输入前端项目地址，一切正常的话那就应该一切正常了哈哈哈

## 注意的问题

当发现访问不到接口或者打不开页面的时候，首先检查一下防火墙！首先云服务器那边控制台的入站规则要放开端口，然后宝塔面板也有一个防火墙，这里面也要放开端口！切记！

![](https://static.yuehaowei.fun/static/blog-images/Linux/11.png)