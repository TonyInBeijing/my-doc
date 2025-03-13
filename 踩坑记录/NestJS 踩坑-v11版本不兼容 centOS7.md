# NestJS 踩坑-v11版本不兼容 centOS7

最近接了个项目，使用 `NesJS v11`开发的后端服务，部署到服务器上发现问题

### 1. NestJS v11 不再支持 v16 & v18

![https://docs.nestjs.com/migration-guide#nodejs-v16-and-v18-no-longer-supported](https://static.yuehaowei.fun/static/blog-images/Linux/12.jpg)

### 2. CentOS 7 glibc 版本不支持 nodejs v18 及以上

![centos7](https://static.yuehaowei.fun/static/blog-images/Linux/13.jpg)

#### 网上很多教程升级 CentOS7 glibc 版本，这里奉劝**慎用**风险太大，可以使用 `Node.js v16` + `NestJS v10` 来开发部署应用 



