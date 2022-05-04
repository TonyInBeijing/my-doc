# NestJS 逃课流攻略（二）项目构建

> 在[上一期](https://blog.yuehaowei.fun/archives/nestjs%E9%80%83%E8%AF%BE%E6%B5%81%E6%94%BB%E7%95%A5%E4%B8%80%E5%87%86%E5%A4%87%E5%BC%80%E5%A7%8B)攻略中，我们成功用脚手架搭起来一个 nestjs 服务，本期我们将继续完善项目结构，并搭建一个 mysql 数据库并通过 nestjs 服务进行增删改查

## 扩军备战

通过脚手架搭建的项目很骨感，并不能满足我们日常使用，所以我们在之前的基础上丰富一下项目结构，让我们做好充足的准备进行后续的开发

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h0oo4g8c5aj20cm0gc3z3.jpg)

按照 nestjs 官网的说明，我们直接用一波教程里的结构，先建一手分类文件夹。

用不用得到以后再说～先建起来就是

![image-20220327193530323](https://tva1.sinaimg.cn/large/e6c9d24ely1h0oo78d4m2j20ee0z63zw.jpg)

分类说明：

- config：放置项目配置文件，比如系统配置，数据库配置等等
- controllers：放置项目所有的控制器文件
- services： 放置项目所有的服务文件
- models：放置项目所有的数据库模型文件
- modules：放置项目所有的模块文件

其他分类大家自己后面看一看文档即可～上面几个很重要～所以说有的课可以逃，有的课还是要上滴～

其实很多前端的小伙伴直接 vue/react 做起手式，压根就没用过这种开发模式，其实如果接触过后端，尤其是 Java 的 Spring MVC，你会发现无比的熟悉～

![image-20220327195020508](https://tva1.sinaimg.cn/large/e6c9d24ely1h0oomo2k51j21880gomyc.jpg)

（灵魂画手就是我😁）

nestjs 都是由一个一个 module 构成的，module里包含各种各样的组件，比如我们的控制器和服务还有一些插件都要在module 里引入

controller，service，model 三者关系嘛，大概就如上图，用户发起一个请求，controller 接收到后，调用 service 来读取数据库，Java 中的 model 负责来连接数据库读取数据，我们这里的 model 用来定义数据库的结构。

## 小试牛刀

对了，这次我们用了 sequelize，这是一个 orm 框架，是用来替代我们写原生 sql 语句的。很多前端同学（包括我在内）sql 平时接触不到，所以我们可以换一种放法来帮我们完成基本的 CRUD 操作

nestjs 封装了一套 sequelize，所以我们直接拿来，这课可以逃

1. 照例先来一波 install

``` bash
$ npm install --save @nestjs/sequelize sequelize sequelize-typescript mysql2
$ npm install --save-dev @types/sequelize
```

2. 首先我们先配置一下数据库连接地址之类的

``` typescript
// src/config/config.ts
class Config {
    readonly dbConfig = {
        host: "192.168.31.133",
        port: 3306,
        database: "nestjs",
        username: "root",
        password: "123456",
    };
}
export default new Config();
```

3. 之前说过，nestjs 都是由 module 构成的，所以我们为数据库单独建一个 module

``` typescript
// src/modules/database.module.ts
import { Module } from "@nestjs/common";
import { SequelizeModule } from "@nestjs/sequelize";
import config from "src/config/config";
import User from "src/models/user/user.model";
const { host, port, username, password, database } = config.dbConfig;

@Module({
    imports: [SequelizeModule.forRoot({
        dialect: "mysql",
        host,
        port,
        username,
        password,
        database,
        models: [User]
    })],
    exports: [SequelizeModule],
})
export default class DatabaseModule {

}
```

nestjs 使用了 `装饰器` 来声明一个 module，就跟 Java 中 Spring Boot 里的`注解`看起来很像，controller 和 service 等其他组件也都是用这种声明方式，既然是逃课流，大家暂时不需要了解其原理，先cc,cv再说

## "M.S.C"

4. 上面引入了一个叫做 User 的 `model` ，代码如下

```typescript
// src/models/user.model.ts

import { AutoIncrement, Column, Model, PrimaryKey, Table } from "sequelize-typescript";

@Table({
    tableName: "user", // 手动定义，告诉 sequelize 数据库中表的名字
    timestamps: false // 设置为false，不然会自动添加一些我们用不到的字段
})
export default class User extends Model {
    @PrimaryKey // 主键
    @AutoIncrement // 自增
    @Column // 字段名
    id: number;

    @Column
    name: string;

    @Column
    sex: number;

    @Column
    createTime: string;
}
```

这个 model 其实就是 mysql 数据库里 User 表的映射，大家看这个大概也就能把表结构猜出个大概吧～

可以看到这个 User 类 继承了 sequelize 的 Model 类，所以它一开始就有了一些方法，比如 `findAll()`,`findOne()`等等，这个我们后面会用到

5. 有了 model，我们需要写一个 service，service 调用 model 和它的一些方法来返回数据库中的数据

```typescript
// src/service/user.service.ts
import { Injectable } from "@nestjs/common";
import { InjectModel } from "@nestjs/sequelize";
import User from "src/models/user/user.model";

@Injectable() // 可注入装饰器,这个装饰器指明它装饰的 UserService 类可以用来依赖注入，这个暂时也不需要了解～
export default class UserService {
    constructor(
        @InjectModel(User) private readonly user: typeof User // 将 User 类注入到 UserService 中
    ) { };
    async findAll(): Promise<User[]> {
        return await this.user.findAll(); // 调用继承自 sequelize 的 Model 类上的方法,返回所有用户
    }
    async findById(id: number): Promise<User> {
        return await this.user.findOne({
            where: {
                id
            }
        }); // 同理，根据 id 返回单个 user 对象
    }
}
```

6. MSC 差个 C，我们补上～

```typescript
// src/controller/user.controller.ts

import { Controller, Get, HttpException, Param } from "@nestjs/common";
import User from "src/models/user/user.model";
import UserService from "src/services/user.service";

@Controller("user") // Controller 装饰器定义一个 controller，路径为 /user
export default class UserController {
    constructor(private readonly userService: UserService) { } // 将 UserService 作为依赖注入
    @Get() // Get 装饰器用来声明下面的方法用 Get 请求，参数为空代表 / 路径
    async findAll(): Promise<User[]> {
        return await this.userService.findAll(); // 请求路径是 http://localhost:8080/user/ 
    };
    @Get(":id")
    async findById(@Param("id") id: string): Promise<User> {
        return await this.userService.findById(id);
    };
}
```



## 融合！

7. 这样 User 模块的 MSC 我们都集齐了，现在可以召唤神龙了～

```typescript
// src/modules/user.module.ts

import { Module } from "@nestjs/common";
import { SequelizeModule } from "@nestjs/sequelize";
import UserController from "src/controllers/user.controller";
import UserService from "src/services/user.service";
import User from "src/models/user/user.model";
@Module({
    imports: [SequelizeModule.forFeature([User])], // User 模块用到的 model，需要用 sequelizeModule 包一下
    controllers: [UserController], // User 模块用到的 Controller 数组
    providers: [UserService], // User 模块用到的依赖（比如我们用 @Injectable 装饰器的 UserService）
})
export default class UserModule {

}
```

8. 最后一步，我们将 Database 模块和 User 模块一起注册到服务中

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import UserModule from './modules/user.module';

import DatabaseModule from './modules/database.module';

@Module({
  imports: [
    DatabaseModule, // 之前使用 sequelize 定义的数据库模块
    UserModule // User 模块
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule { }
```

## 测试

我们打开终端，cd 到项目的目录并启动项目

```bash
$ npm run start:dev
```

然后打开浏览器，在地址栏中输入 `http://localhost:8080/user`

![image-20220327210109449](https://tva1.sinaimg.cn/large/e6c9d24ely1h0oqoclwm4j21bo0u00uz.jpg)

(这里我用了 postman，一个很好用的接口测试工具，推荐大家使用～)

可以看到我们的接口顺利给我们返回了数据～开心～

## 总结

通过以上步骤，我们成功引入了 sequelize，并使用它连上了我们的 mysql 数据库，乍一看感觉好绕，可能会觉得还不如直接用 express/koa 梭哈，我再给大家捋捋啊～

添加一个模块，分以下几步：

1. 建一个 model，用来映射数据库表结构，告诉 sequelize 数据库表的字段，这时候 model 可能是多个，大家一个一个来定义就好～
2. 建一个 service，在构造函数中引入 model，就可以用 sequelize 的方法 CRUD 数据库了
3. 建一个 controller，并定义好路径，用来响应用户的请求，在构造函数中引入 service，这条线就穿起来了
4. 建一个 module，把 controller，service 引入，用 SequelizeModule.forFeature([]) 来引入上面定义的 model
5. 在 `app.module.ts` 中引入新建的 module，这样就齐活儿了～

这时候有人可能会问，为啥 controller 必须通过 service 来连接 model，其实正常来讲，代码怎么写完全是咱们自己自由，只不过等后面功能比较复杂了，我们可能习惯于在 controller 中处理业务逻辑，而在 service 中处理数据库逻辑，这个是大家普遍的习惯～不用过多纠结到底哪部分代码写在哪里～

其实 nestjs 默认是基于 express 构建的项目，它帮我们实现了很多基础功能，并给我们制定了一套开发规范，作为一个 .net 起手，中途写过一段 Java 的前端程序员，我很希望 nodejs 也有一个像 Java Spring 或者 .net MVC 这样的框架，我认为这是 nodejs 作为主流后台开发语言的必经之路吧。。（哈哈哈哈算了我瞎说的😁）

修完福报赶紧补一篇博客，然后这时候终于可以去当我的艾尔登之王了！大家下期再见～