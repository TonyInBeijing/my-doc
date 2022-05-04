# NestJS 逃课流攻略（一）准备开始

> 近期《艾尔登法环》游戏大火，宫崎老贼又一次跌上了神坛，我也是沉迷其中不能自拔，由于难度过高的关卡设计导致全网研究“逃课流”打法，能润就润，绝不坐牢，我也是逃了不少的课，目前终于是打过了拉尼老婆的支线🎉🎉
>
> 这一个系列的主题也主要是“逃课”，NestJS 的英文文档着实对我三脚猫英文水平提出了考验，可以说是出生点门口的大树守卫啊，对想上手 NestJS 的小伙伴来说极不友好，所以本着能润就润的原则我会尽量简洁明了地记录一下我的 NestJS 一周目也就是开荒体验，既然是开荒，那么一定有一些失误和不足，恳请大家见谅，如有可能望大家不吝赐教～

## 引导之始

> NestJS 官网：https://nestjs.com/

首先我们需要配置好我们的开发环境，由于是使用nodejs做服务端，所以需要在你的电脑上安装一些必要的工具

- nodejs
- npm/yarn
- 数据库可视化工具，这里我用的是 navicat（如果你确定你不需要，那么大神可以略过这一条）
- mysql 数据库，这里我在本地测试机装了一个 mysql 服务，你也可以用云服务器或云数据库
- 一个你喜欢的IDE，比如 vscode，当然你要是用 vim，我们可以交个朋友

贴一下我的环境

```bash

# admin @ Mac-mini-M1 in ~/Documents/nestjs-project on git:main o [21:48:50] 
$ node -v
v16.10.0
(base) 
# admin @ Mac-mini-M1 in ~/Documents/nestjs-project on git:main o [21:48:52] 
$ npm -v
7.24.0
(base) 
```

NestJS 官方提供了一个脚手架来构建项目，我们先安装一波：

```bash
$ npm install @nestjs/cli -g
```

同步一下版本号

```bash
$ nest -v
8.2.4
```

然后找一个喜欢的文件路径，使用脚手架构建一下项目

```bash
$ nest new nestjs-project # nestjs-project是项目名称，你可以改成你喜欢的～
```

这里会让你选一下包管理工具，我因为有梯子就默认选了 npm，推荐使用 yarn，因为它更快

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0j0crggwqj212y0u0ah9.jpg)

稍作等待，我们就可以看到构建成功的界面了，然后运行

```bash
$ cd nestjs-project
$ npm run start
```

打开浏览器，在地址栏里输入 http://localhost:3000,一切正常的话就会看到 Hello World 

恭喜你正式进入游戏🎉🎉

## 开始冒险

开始之前我们先看一下刚才构建出来的项目结构

![image-20220322223917392](https://tva1.sinaimg.cn/large/e6c9d24ely1h0j1ex4e20j20ek0oojsb.jpg)

大家伙不要被这些个东西唬到啊，都是小场面～

我们先找我们熟悉的 `package.json` 文件看一下,这里我贴一下源文件，以防以后被依赖版本搞懵

```json
// package.json
{
  "name": "nestjs-project",
  "version": "0.0.1",
  "description": "",
  "author": "",
  "private": true,
  "license": "UNLICENSED",
  "scripts": {
    "prebuild": "rimraf dist",
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  },
  "dependencies": {
    "@nestjs/common": "^8.0.0",
    "@nestjs/core": "^8.0.0",
    "@nestjs/platform-express": "^8.0.0",
    "reflect-metadata": "^0.1.13",
    "rimraf": "^3.0.2",
    "rxjs": "^7.2.0"
  },
  "devDependencies": {
    "@nestjs/cli": "^8.0.0",
    "@nestjs/schematics": "^8.0.0",
    "@nestjs/testing": "^8.0.0",
    "@types/express": "^4.17.13",
    "@types/jest": "27.4.1",
    "@types/node": "^16.0.0",
    "@types/supertest": "^2.0.11",
    "@typescript-eslint/eslint-plugin": "^5.0.0",
    "@typescript-eslint/parser": "^5.0.0",
    "cz-conventional-changelog": "^3.3.0",
    "eslint": "^8.0.1",
    "eslint-config-prettier": "^8.3.0",
    "eslint-plugin-prettier": "^4.0.0",
    "jest": "^27.2.5",
    "prettier": "^2.3.2",
    "source-map-support": "^0.5.20",
    "supertest": "^6.1.3",
    "ts-jest": "^27.0.3",
    "ts-loader": "^9.2.3",
    "ts-node": "^10.0.0",
    "tsconfig-paths": "^3.10.1",
    "typescript": "^4.3.5"
  },
  "jest": {
    "moduleFileExtensions": [
      "js",
      "json",
      "ts"
    ],
    "rootDir": "src",
    "testRegex": ".*\\.spec\\.ts$",
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    "collectCoverageFrom": [
      "**/*.(t|j)s"
    ],
    "coverageDirectory": "../coverage",
    "testEnvironment": "node"
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
}

```

这里面有一个 `config`配置

在 `devdependencies`下还有一个 `"cz-conventional-changelog": "^3.3.0"`

这个是跟项目开发没关系的依赖，但是我是要着重推荐一波的，这个东西叫 commitizen ，是帮助我们在提交到GitHub或者其他代码仓库时，规范我们的 commit 的，由于篇幅的原因啊，在此不做过多说明，GitHub 地址在[这里](https://github.com/commitizen/cz-cli)，大家有空可以看一看，不过没关系啊不会影响到我们开发

这个时候呢，我们可以先把正在运行的终端停下来，因为 NestJS 提供了一个热更新的选项，我们修改完代码后会自动帮我们重启服务，还是非常的贴心呀～

```bash
$ npm run start:dev
```

这时候就开启热更新啦～

下一篇会跟大家着重分析一波`src`里的各个文件，开荒先要点亮地图才知道去哪里对吧～由于福报太多，一时半会修不完，我们下期再见～

撒呦哪啦！