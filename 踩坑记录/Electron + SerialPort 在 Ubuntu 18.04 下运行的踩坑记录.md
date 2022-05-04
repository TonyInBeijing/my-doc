# Electron + SerialPort 在 Ubuntu 18.04 下运行的踩坑记录

> 在之前 [《Electron + React 全家桶 开发到打包踩坑记录》](https://tonyxx.gitbook.io/main-page/v/ri-chang-cai-keng/)中记录了开发+打包过程中遇到的坑和填坑方法，最终完结撒花🎉
>
> 但是当打包后的程序在 Ubuntu 18.04 上运行时又出现了新的问题，看来花还是撒早了 5555...
>
> 经过一番研究，主要问题集中在 electron、serialport 和 ubuntu 18.04 版本不匹配
>
> 以下使用我自己的 ubuntu 18.04 开发机

## 项目依赖

- ubuntu 18.04 LTS
- electron v14.1.1
- serialport v9.2.1

## 开始打包

运行打包命令

```bash
# 先打包 react
npm run build
# 再打包 electron-builder 打包 Linux deb 安装包
sudo electron-builder --linux deb --x64
```

终端中看到一条：

```bash
To ensure your native dependencies are always matched electron version, simply add script `"postinstall": "electron-builder install-app-deps" to your `package.json`
```

这句话的意思是 serialport 是一个 c++ 原生模块，需要为它单独安装所需要的依赖来匹配 electron 的版本,这里由于我的

electron-builder 是全局安装的，所以先不需要修改 package.json ，我们先手动运行一下这个命令

```bash
# 删除刚才打包路径
sudo rm -rf dist
# 安装原生模块依赖
sudo electron-builder install-app-deps 
# 重新打包
sudo electron-builder --linux deb
```

## 大坑出现

打包完成后安装并运行程序，发现程序运行白屏，打开开发者工具发现报错如下

![2022-02-16 21-30-33屏幕截图](https://tva1.sinaimg.cn/large/e6c9d24ely1gzfojdkt8gj20ua0kqjy9.jpg)

根据报错信息发现没有找到 GLIBC_2.28

我们查看一下本机的 GLIBC 版本

```bash
ldd --version
```

![2022-02-16 21-34-51屏幕截图](https://tva1.sinaimg.cn/large/e6c9d24ely1gzfoktzrj8j21a208e0ua.jpg)

发现版本号是 2.27，ubuntu 18.04 版本默认支持到 2.27版本，所以需要升级一下 glibc 

## 版本匹配

这里需要注意一个问题： glibc 升级有破坏 ubuntu 系统的风险，所以现在有2个解决方案：

- 使用 ubuntu 自带的软件更新升级到 20.04 版本
- 降级 electron 和 serialport 到合适的版本

首先由于项目的特殊性，我们的工程机系统最好不要升级，否则可能出现其他新的问题，所以我只能选择第二个解决方案，所以如何选择 electron，serialport 的版本和 ubuntu 18.04 匹配是关键，经过一顿寻找后发现了一位大兄弟的评论帮助很大 [serialport 所需的 glibc 版本](https://github.com/serialport/node-serialport/issues/2266)

![image-20220216215408030](https://tva1.sinaimg.cn/large/e6c9d24ely1gzfp1gexvnj21gq0ek75u.jpg)

由于 ubuntu 18.04 glibc 默认最高支持到 2.27,所以选择 serialport@9.0.7 版本，但是由于打包时需要对原生模块进行编译，serialport包含了一个 serialport/bindings 的依赖，编译时 electron-builder install-app-deps 需要下载对应 electron 版本的压缩文件

 [serialport/bindings@9.0.7](https://github.com/serialport/node-serialport/releases/tag/%40serialport%2Fbindings%409.0.7) GitHub 下载地址

![image-20220216220037984](https://tva1.sinaimg.cn/large/e6c9d24ely1gzfp880w4dj20x40u0dlq.jpg)

我们在这里面找 electron 对应的压缩包（作者提供了 electron 和 node 两个版本的，我们只需要找 electron 版本的）

后面 v47 指的是electron 的 ABI 版本号。 什么是 [ABI](https://nodejs.org/zh-cn/docs/guides/abi-stability/)？

有一个查看 electron 和 nodejs ABI 版本号的包： [node-abi](https://github.com/electron/node-abi)

这里新建一个项目，方便以后也用到

```bash
cd ~
mkdir abi-tool
npm init
# 一路回车就行
npm install node-abi
touch index.js
vim index.js
```

```javascript
// index.js

const nodeAbi = require('node-abi');
// 通过上图 serialport/bindings@9.0.7 ，选择支持的最新的 electron，ABI 版本号为 85
// 我们这里获取一下 ABI 版本号为 85 的 electron 版本号
console.log(nodeAbi.getTarget('85', 'electron'));
// '11.0.0'
```

所以接下来我们安装 electron@11.0.0 和 serialport@9.0.7

```bash
# 安装 electron
npm install electron@11.0.0 --save-dev
# 安装 serialport
npm install serialport@9.0.7
# 更新一下 @serialport/bindings，上面命令不会将 bindings 降级，这里需要手动降一下
npm install @serialport/bindings@9.0.7
```

然后删除之前的打包文件，重新运行打包命令

```bash
sudo rm -rf build dist
npm run build
sudo electron-builder install-app-deps
sudo electron-builder --linux deb
```

## 完结撒花🎉（又撒一次...）

这个时候重新安装 .deb 发现运行正常！填坑完成！（暂时...）

本文主要记录了作为一个 Linux 萌新的交学费之路，各位大佬如果有更好的解决方案恳请赐教～