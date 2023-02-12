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

