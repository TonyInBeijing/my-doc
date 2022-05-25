# NestJS 逃课流攻略（三）增删改查

> 好久不见！这两个月并没有偷偷摸鱼，奈何福报太多在小程序&app&NestJS&各种奇怪的需求讨论会中反复横跳，博客荒废了许久，要好好检讨！不过在这两个月的实战中奇怪的知识又增加了（不是），以后会考虑单开一波 [uni-app](https://uniapp.dcloud.io/) 开发微信小程序+[React Native](https://reactnative.dev/) 开发安卓/IOS app 的专栏博客，希望不要只是一个 flag 真的真的福报太多了555...

## 1.准备工作

![image-20220327193530323](https://tva1.sinaimg.cn/large/e6c9d24ely1h0oo78d4m2j20ee0z63zw.jpg)



根据上一期建好的项目结构，我们现在需要做的就是增加一个商铺模块来进行商铺的增删改查操作

## 2.上代码～

```typescript
/*
 * @Author: TonyInBeijing
 * @Date: 2022-04-05 22:39:48
 * @LastEditors: TonyInBeijing
 * @LastEditTime: 2022-04-06 23:43:40
 * @FilePath: /nestjs-project/src/modules/shop.module.ts
 * @Description: 商户模块
 * 
 */

import { Module } from "@nestjs/common";
import { SequelizeModule } from "@nestjs/sequelize";
import ShopControlelr from "src/controllers/shop.controller";
import Shop from "src/models/shop/shop.model";
import ShopService from "src/services/shop.service";

@Module({
    imports: [
        SequelizeModule.forFeature([Shop])
    ],
    controllers: [ShopControlelr],
    providers: [ShopService]
})
export default class ShopModule {

}
```

上面代码中涉及到了一些依赖文件，shopController、ShopModel、ShopService 分别都是上期博客中介绍的 M.S.C架构，此处不多阐述，毕竟还是逃课流攻略，先照着抄一波~

需要注意的是`@Module()`装饰器里的`imports`，它是一个数组，包含了这个模块需要引入的`模块`，这里只引用了一个 SequelizeModule.forFeature([Shop])，这个模块的作用是将 Sequelize 包含的 Shop Model 暴露给 Shop 模块使用，这样说可能有点绕，不急接着往下看~

```typescript
/*
 * @Author: TonyInBeijing
 * @Date: 2022-04-05 22:40:00
 * @LastEditors: TonyInBeijing
 * @LastEditTime: 2022-04-13 00:01:19
 * @FilePath: /nestjs-project/src/controllers/shop.controller.ts
 * @Description: 
 * 
 */

import { Controller, Get, Param } from "@nestjs/common";
import Shop from "src/models/shop/shop.model";
import ShopService from "src/services/shop.service";


@Controller("shop")
export default class ShopControlelr {
    constructor(
        private readonly shopService: ShopService
    ) { }

    /**
     * @description 获取店铺列表
     * @returns 
     */
    @Get()
    async findAll(): Promise<Shop[]> {
        return await this.shopService.findAll();
    }

    /**
     * @description 通过店铺id获取店铺信息
     * @param id 
     * @returns 
     */
    @Get(":id")
    async findOneById(@Param("id") id: number): Promise<Shop> {
        return await this.shopService.findOneById(id);
    }
}
```

这是 shop 模块的 controller 文件，上面有一些 NestJS 实现的装饰器，乍一看特别像 springboot 的注解，javascript 的装饰器具体可以看[这一篇](https://segmentfault.com/a/1190000014495089) ，大佬介绍的非常详细，有空我也可以来一篇十分简单的介绍～这里暂时先记下这几个常用装饰器的作用吧～

- @Controller("A") ：这个装饰器用来声明下面的类是一个 NestJS 的 Controller，每一个 Controller 都需要在类名上加上这个注解，这样 NestJS 可以识别到这个控制器
- @Get("B")：这个装饰器用于声明一个 Get 请求，括号中写请求的地址，这样用户请求 http://localhost:9090/A/B 就可以访问到这个接口，相应的还有 @Post 等装饰器，都是用来声明请求类型的。这个参数可以写成 `:id` 这种形式的，方便咱们构建 Restful 风格接口 
- @Param("id")：这个装饰器是用来获取路由参数的，比如上面 `findOneById`这个方法，假如`id=1`，那么请求路径是 http://localhost:9090/shop/1，这时使用@Param("id") id,就能在下面的方法中拿到 id = 1。类似的还有@Query("id")这种装饰器，是用来取 Get 请求中 ? 后面参数的值的，比如 http://localhost:9090/shop?id=1这种风格的请求路径

```typescript
/*
 * @Author: TonyInBeijing
 * @Date: 2022-04-05 22:40:10
 * @LastEditors: TonyInBeijing
 * @LastEditTime: 2022-04-07 23:10:18
 * @FilePath: /nestjs-project/src/services/shop.service.ts
 * @Description: 
 * 
 */

import { Injectable } from "@nestjs/common";
import { InjectModel } from "@nestjs/sequelize";
import Shop from "src/models/shop/shop.model";


@Injectable()
export default class ShopService {
    constructor(
        @InjectModel(Shop) private readonly shop: typeof Shop
    ) { }

    /**
     * @description 获取店铺列表
     * @returns 
     */
    async findAll(): Promise<Shop[]> {
        return await this.shop.findAll();
    }

    /**
     * @description 根据 id 获取店铺信息
     * @param id 
     * @returns 
     */
    async findOneById(id: number): Promise<Shop> {
        return await this.shop.findOne({
            where: {
                id
            }
        });
    }
}
```

Shop.Service的作用在我的理解里是用来与数据库交互的，虽然上面的代码里并没有直接显示 sql 语句，但是我们通过 `Model`层与数据库建立了映射，`Service`层引入了`Model`，而 `Controller`层又引入了`Service` ，这样就可以通过请求来进行数据库的增删改查了

首先通过 @Injectable() 装饰器来声明这是一个“可注入”的类，使用了这个装饰器，我们可以在`Shop.Module`的`provider`数组中加入这个Service，这样就可以被模块访问到，在这个Service 类的构造函数中，我们需要使用 `@InjectModel(Shop)`来把我们定义好的`Shop.Model`再引入到 Service 中，这样我们就可以使用 NestJS 定义好的一些与数据库交互的方法了，比如上面的 shop.findAll() 就是查询 shop 表中所有的数据,具体的 api 可以看[这里](https://www.sequelize.com.cn/)

```typescript
/*
 * @Author: TonyInBeijing
 * @Date: 2022-04-06 23:38:56
 * @LastEditors: TonyInBeijing
 * @LastEditTime: 2022-04-11 23:30:16
 * @FilePath: /nestjs-project/src/models/shop/shop.model.ts
 * @Description: 
 * 
 */

import { AutoIncrement, Column, Model, PrimaryKey, Table } from "sequelize-typescript";

@Table({
    tableName: "shop",
    timestamps: false
})
export default class Shop extends Model {
    @PrimaryKey
    @AutoIncrement
    @Column
    id: number;

    @Column
    name: string;

    @Column
    rank: number;

    @Column
    createTime: string;

    @Column
    status: number;
}

```

上面的代码是我们根据数据库使用js来建立的表结构，首先声明一个 Shop 类，这个类名不一定与数据库中的表名一致，然后这个类继承自 NestJS 的 Model 类，它提供了一系列与数据库交互的方法，省去我们许多麻烦

@Table() 装饰器声明了这个类是一个“表”，确实直接翻译过来就是最恰当的意思哈哈哈哈，参数常用的就这两个：

- tableName：定义数据库中我们真正的表名，这样 sequelize 可以根据表名找到数据库中的表
- timestamps：这个选项很重要！在使用 sequelize 定义好的方法时（例如 findAll() 这种），它会自动给我们生成 sql 语句，这些语句中会包含一些它默认的字段，比如 UpdateAt，CreateAt这种，但是我们一般在表中会定义不同的字段，所以这里设置成`false`，方便我们手动声明需要更新的字段，说白了就是怕它添乱～

## 3.将 shop 模块挂载到项目中

上面我们已经定义好了一个基本完整的业务模块，这时候在浏览器或者 postman 中并不能访问我们定义的接口，还需要在 app.module.ts 中引入我们的 shop.module 这样用户才能访问我们的接口

```typescript
/*
 * @Author: TonyInBeijing
 * @Date: 2022-03-21 23:04:26
 * @LastEditors: TonyInBeijing
 * @LastEditTime: 2022-04-23 22:09:41
 * @FilePath: /nestjs-project/src/app.module.ts
 * @Description: 
 * 
 */
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

import DatabaseModule from './modules/database.module';
import ShopModule from './modules/shop.module';


@Module({
  imports: [
    DatabaseModule,
    ShopModule
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule { }
```

将 ShopModule 加入到 imports 数组中，然后我们刷新一下或者在命令行运行一下启动命令：`npm run start:dev`

## 4.测试一波～

由于我们只定义了一些 Get 请求，这时我们只需打开浏览器，在地址栏中输入接口地址即可～

![image-20220526011506504](https://tva1.sinaimg.cn/large/e6c9d24ely1h2l5kt1ylpj21mq0jo76c.jpg)

恭喜你成功在项目中增加了一个模块，只不过现在表中没有数据，所以返回了空数组

至于它为什么会返回一个包含 code，message，data 的对象，这个正是下面需要说的`拦截器`和`过滤器` ，下期揭晓～

## 5.总结

其实加入一个模块没有很复杂，大体流程如下：

1. 创建一个 xxx.module.ts，xxx.controller.ts, xxx.service.ts, xxx.model.ts 然后从后往前给它们套起来～
2. 将 xxx.model.ts 加入到 database.module.ts 中，就是上一期我们定义了那个数据库模块，里面包含了我们这个项目所有的需要用的表
3. 然后将 xxx.module.ts 加入到 app.module.ts中，重启项目即可～

这波实在是鸽了太久太久，这里由衷说声抱歉。。我知道虽然没有很多人看，但是吧，作为激励自己进步的一种方式，以后还是会抓紧写博客～争取今年能写够50篇吧！这个 flag 不算太离谱，加油！