Egg.js 由koa.js扩展而来，他参考了 Ruby on Rails 的设计哲学，以约定优先的配置。

- app/router.js，路由映射配置
- app/controller, 用来存放控制器的目录，处理跳转相关
- app/service, 用来存放业务逻辑
- app/middleware,用来存放中间件的目录
- app/public,存放静态文件的目录
- app/extends, 扩展框架目录，比如往ctx上添加一些变量方法等
- config, 配置目录，中间的配置项，环境变量的配置
- test,测试文件目录

# egg简单实现

编写controller

```js
			// app/controller/home.js
const Controller = require('egg').Controller;

class HomeController extends Controller {
  async index() {
    this.ctx.body = 'Hello world';
  }
}

module.exports = HomeController;
```

编写路由

```js
// app/router.js
module.exports = app => {
  const { router, controller } = app;
  router.get('/', controller.home.index);
};
```

增加配置文件

```js
// config/config.default.js
exports.keys = <此处改为你自己的 Cookie 安全字符串>;			
```

完善package.json

```json
{
  "name": "egg-example",
  "scripts": {
    "dev": "egg-bin dev"
  }
}
```

此时目录结构如下

```
egg-example
├── app
│   ├── controller
│   │   └── home.js
│   └── router.js
├── config
│   └── config.default.js
└── package.json
```

# 基础对象

## 内置对象

### 全局应用对象application

**描述：** 在一个应用中，只会实例化一个。可以在上面挂载一些全局方法或属性，也可以在上面注册事件。

**使用：**

```js
// app.js
module.exports = app => {
  app.once('server', server => {
  });
  app.on('error', (err, ctx) => {
  });
  app.on('request', ctx => {
  });
  app.on('response', ctx => {
  });
};
```

**获取方式:**

- 自定义脚本 app.js:  module.exports = app => {...};
- Controller文件：this.ctx.app
- 在继承Controller或者Service基类的实例中：this.app



### 请求级别对象context

**描述：**每一次收到到请求，就会实例化一个context对象，这个对象封装了用户的请求信息，会将所有的service挂载上去。

**使用：**

```js
// app.js
module.exports = app => {
  app.beforeStart(async () => {
    const ctx = app.createAnonymousContext();
    // preload before app start
    await ctx.service.posts.load();
  });
}
```

**获取方式：**

- 在继承Controller或者Service基类的实例中：this.app
- 在Middleware中通过形参ctx：async  function middleware(ctx, next) {....}
- 在定时任务schedule中通过形参：async ctx => {...}
- 在非请求时通过application手动创建：app.createAnonymousContext();

### Request & Response

**描述：**请求级别对象，分别提供了一些辅助方法来获取请求参数和设置相应参数

**获取方式：**通过Context.request和Context.response

**使用：**

```js
// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    const { app, ctx } = this;
    const id = ctx.request.query.id;
    ctx.response.body = app.cache.get(id);
  }
}
```

- [Koa](http://koajs.com/) 会在 Context 上代理一部分 Request 和 Response 上的方法和属性，参见 [Koa.Context](http://koajs.com/#context)。
- 如上面例子中的 `ctx.request.query.id` 和 `ctx.query.id` 是等价的，`ctx.response.body=` 和 `ctx.body=` 是等价的。
- 需要注意的是，获取 POST 的 body 应该使用 `ctx.request.body`，而不是 `ctx.body`。

### Controller

描述：框架提供了一些基类，并推荐所有controller来继承。它拥有以下几列属性

- ctx-当前请求的Context实例
- app-应用的Application实例
- config-应用的配置
- service-应用的所有service
- logger-为当前controller封装的logger对象

使用：

```js
// app/controller/user.js

// 从 egg 上获取（推荐）
const Controller = require('egg').Controller;
class UserController extends Controller {
  // implement
}
module.exports = UserController;
```

###  Service

描述：同controller,属性也同

### Helper

**描述：**Helper用来提供一些使用的util函数。它可以将一些常用的动作抽离在helper.js里面成为一个独立的函数。主要是helper实例可以在模板中获得。

**获取：**Context.helper

**自定义helper方法：**

```js
// app/extend/helper.js
module.exports = {
  formatUser(user) {
    return only(user, [ 'name', 'phone' ]);
  }
};
```

### Config

**描述：**将一些需要硬编码的业务配置放到配置文件里，同时也支持不同的运行环境使用不同的配置

**获取方式：**

- Applicition.config
- 在Controller,Service,Helper中，this.config

### Logger

**描述：**打印各种级别的日志到对应的日志文件里面,每个logger对象提供了4个级别的方法

- logger.debug()
- logger.info()
- logger.warn()
- logger.error()

框架中提供了多个logger对象

- App logger
- App coreLogger
- Context logger:打印的日志前面会带请求信息相关
- Context coreLogger:一般是插件或框架用
- Controller logger & Service logger：比contex logger 多打印文件路径



### Subscription

**描述：**订阅模型是一种比较常见的开发模式，譬如消息中间件的消费者或调度任务。因此我们提供了 Subscription 基类来规范化这个模式。

**使用：**

```js
const Subscription = require('egg').Subscription;
class Schedule extends Subscription {
  // 需要实现此方法
  // subscribe 可以为 async function 或 generator function
  async subscribe() {}
}
```

## 配置

后加载配置的会覆盖掉先加载的同名配置

```js
-> 插件 config.default.js
-> 框架 config.default.js
-> 应用 config.default.js
-> 插件 config.prod.js
-> 框架 config.prod.js
-> 应用 config.prod.js
```

- 按插件 => 框架 => 应用依次加载
- 插件之间的顺序由依赖关系决定，被依赖方先加载，无依赖按 object key 配置顺序加载，具体可以查看[插件章节](https://eggjs.org/zh-cn/advanced/plugin.html)
- 框架按继承顺序加载，越底层越先加载。



## 中间件（Middleware）

**描述：**基于洋葱圈模型,在框架中，一个完整的中间件是包含了配置处理的。我们约定一个中间件是一个放置在 `app/middleware` 目录下的单独文件，它需要 exports 一个普通的 function，接受两个参数：

- options: 中间件的配置项，框架会将 `app.config[${middlewareName}]` 传递进来。
- app: 当前应用 Application 的实例。

**使用：**

1.**编写中间件**

```js
// app/middleware/gzip.js
const isJSON = require('koa-is-json');
const zlib = require('zlib');

module.exports = options => {
  return async function gzip(ctx, next) {
    await next();

    // 后续中间件执行完成后将响应体转换成 gzip
    let body = ctx.body;
    if (!body) return;

    // 支持 options.threshold
    if (options.threshold && ctx.length < options.threshold) return;

    if (isJSON(body)) body = JSON.stringify(body);

    // 设置 gzip body，修正响应头
    const stream = zlib.createGzip();
    stream.end(body);
    ctx.body = stream;
    ctx.set('Content-Encoding', 'gzip');
  };
};
```

**2.当中间件编写完成后，还需要手动挂载：支持一下方式：**

1. 在应用中使用：在config.default.js总增加如下配置

   ```js
   module.exports = {
     // 配置需要的中间件，数组顺序即为中间件的加载顺序
     middleware: [ 'gzip' ],
   
     // 配置 gzip 中间件的配置
     gzip: {
       threshold: 1024, // 小于 1k 的响应体不压缩
     },
   };
   ```

2. 在框架和插件中使用，需要在app.js 中添加

   ```js
   // app.js
   module.exports = app => {
     // 在中间件最前面统计请求时间
     app.config.coreMiddleware.unshift('gizp');
   };
   ```

3. router中使用，可直接在app/router.js中实例化和挂载

   ```js
   module.exports = app => {
     const gzip = app.middleware.gzip({ threshold: 1024 });
     app.router.get('/needgzip', gzip, app.controller.handler);
   };
   ```

**使用Koa中间件：**在框架中可以很容易引入Koa中间件生态，如果不符合，自行处理下。

```js
// app/middleware/compress.js
// koa-compress 暴露的接口(`(options) => middleware`)和框架对中间件要求一致
module.exports = require('koa-compress');
```

```js
// config/config.default.js
module.exports = {
  middleware: [ 'compress' ],
  compress: {
    threshold: 2048,
  },
};
```

**通用配置：**在config中设置

- enable：控制中间件是否开启。
- match：设置只有符合某些规则的请求才会经过这个中间件。
- ignore：设置符合某些规则的请求不经过这个中间件。

备注：match和ignore不允许同时配置，他们都支持1.字符串，2.正则，3函数：实参为Context

```js
module.exports = {
  gzip: {
    match(ctx) {
      // 只有 ios 设备才开启
      const reg = /iphone|ipad|ipod/i;
      return reg.test(ctx.get('user-agent'));
    },
  },
};
```



## router路由

Router 主要用来描述请求 URL 和具体承担执行动作的 Controller 的对应关系

### 如何定义router

```js
router.verb('path-match', app.controller.action);
router.verb('router-name', 'path-match', app.controller.action);
router.verb('path-match', middleware1, ..., middlewareN, app.controller.action);
router.verb('router-name', 'path-match', middleware1, ..., middlewareN, app.controller.action);
```

- verb - 用户触发动作，支持 get，post 等所有 HTTP 方法;
- router-name 给路由设定一个别名，可以通过 Helper 提供的辅助函数 `pathFor` 和 `urlFor` 来生成 URL。(可选)
- path-match - 路由 URL 路径。
- middleware1 - 在 Router 里面可以配置多个 Middleware。(可选)
- controller - 指定路由映射到的具体的 controller 上，controller 可以有两种写法：
  - `app.controller.user.fetch` - 直接指定一个具体的 controller
  - `'user.fetch'` - 可以简写为字符串形式

### RESTful 风格的 URL 定义

```js
// app/router.js
module.exports = app => {
  const { router, controller } = app;
  router.resources('posts', '/api/posts', controller.posts);
  router.resources('users', '/api/v1/users', controller.v1.users); // app/controller/v1/users.js
};
```

上面代码就在 `/posts` 路径上部署了一组 CRUD 路径结构，对应的 Controller 为 `app/controller/posts.js` 接下来， 你只需要在 `posts.js` 里面实现对应的函数就可以了。

| Method | Path            | Route Name | Controller.Action             |
| ------ | --------------- | ---------- | ----------------------------- |
| GET    | /posts          | posts      | app.controllers.posts.index   |
| GET    | /posts/new      | new_post   | app.controllers.posts.new     |
| GET    | /posts/:id      | post       | app.controllers.posts.show    |
| GET    | /posts/:id/edit | edit_post  | app.controllers.posts.edit    |
| POST   | /posts          | posts      | app.controllers.posts.create  |
| PUT    | /posts/:id      | post       | app.controllers.posts.update  |
| DELETE | /posts/:id      | post       | app.controllers.posts.destroy |

```
// app/controller/posts.js
exports.index = async () => {};

exports.new = async () => {};

exports.create = async () => {};

exports.show = async () => {};

exports.edit = async () => {};

exports.update = async () => {};

exports.destroy = async () => {};
```

### 参数获取

#### Query String 方式

```js
// app/router.js
module.exports = app => {
  app.router.get('/search', app.controller.search.index);
};

// app/controller/search.js
exports.index = async ctx => {
  ctx.body = `search: ${ctx.query.name}`;
};

// curl http://127.0.0.1:7001/search?name=egg
```

#### 参数命名方式

```js
// app/router.js
module.exports = app => {
  app.router.get('/user/:id/:name', app.controller.user.info);
};

// app/controller/user.js
exports.info = async ctx => {
  ctx.body = `user: ${ctx.params.id}, ${ctx.params.name}`;
};

// curl http://127.0.0.1:7001/user/123/xiaoming
```

#### 复杂参数的获取

```js
// app/router.js
module.exports = app => {
  app.router.get(/^\/package\/([\w-.]+\/[\w-.]+)$/, app.controller.package.detail);
};

// app/controller/package.js
exports.detail = async ctx => {
  // 如果请求 URL 被正则匹配， 可以按照捕获分组的顺序，从 ctx.params 中获取。
  // 按照下面的用户请求，`ctx.params[0]` 的 内容就是 `egg/1.0.0`
  ctx.body = `package:${ctx.params[0]}`;
};

// curl http://127.0.0.1:7001/package/egg/1.0.0
```

#### 表单内容获取

```js
// app/router.js
module.exports = app => {
  app.router.post('/form', app.controller.form.post);
};

// app/controller/form.js
const createRule = {
  username: {
    type: 'email',
  },
  password: {
    type: 'password',
    compare: 're-password',
  },
};
exports.post = async ctx => {
  // 如果校验报错，会抛出异常
  ctx.validate(createRule);
  ctx.body = `body: ${JSON.stringify(ctx.request.body)}`;
};

// 模拟发起 post 请求。
// curl -X POST http://127.0.0.1:7001/form --data '{"name":"controller"}' --header 'Content-Type:application/json'
```

注意，表单内容须进行表单校验，防止csrf

### 重定向

```js
app.router.redirect('/', '/home/index', 302);//内部重定向
ctx.redirect(`http://cn.bing.com/search?q=${q}`);//外部重定向
```

# Controller控制器

框架推荐 Controller 层主要对用户的请求参数进行处理（校验、转换），然后调用对应的 [service](https://eggjs.org/zh-cn/basics/service.html) 方法处理业务，得到业务结果后封装并返回：

## Service服务

业务处理

## 插件

### 为什么要是使用插件？

1. 中间件加载其实是有先后顺序的，但是中间件自身却无法管理这种顺序，只能交给使用者。这样其实非常不友好，一旦顺序不对，结果可能有天壤之别。
2. 中间件的定位是拦截用户请求，并在它前后做一些事情，例如：鉴权、安全检查、访问日志等等。但实际情况是，有些功能是和请求无关的，例如：定时任务、消息订阅、后台逻辑等等。
3. 有些功能包含非常复杂的初始化逻辑，需要在应用启动的时候完成。这显然也不适合放到中间件中去实现。

### 中间件，插件，应用之间的关系

一个插件其实就是一个『迷你的应用』，和应用（app）几乎一样：

- 它包含了 [Service](https://eggjs.org/zh-cn/basics/service.html)、[中间件](https://eggjs.org/zh-cn/basics/middleware.html)、[配置](https://eggjs.org/zh-cn/basics/config.html)、[框架扩展](https://eggjs.org/zh-cn/basics/extend.html)等等。
- 它没有独立的 [Router](https://eggjs.org/zh-cn/basics/router.html) 和 [Controller](https://eggjs.org/zh-cn/basics/controller.html)。
- 它没有 `plugin.js`，只能声明跟其他插件的依赖，而**不能决定**其他插件的开启与否

### 使用

1. 通过npm包引入

2. 在应用或框架的config/plugin.js中声明

   ```js
   export.mysql = {
   	enable: true,//是否开启
   	package: 'egg-mysql',//模块名称
   }
   ```

3. 就可以直接使用了 app.mysql

4. 配置可以在config.default.js中覆盖

   ```js
   exports.mysql = {
     client: {
       host: 'mysql.com',
       port: '3306',
       user: 'test_user',
       password: 'test_password',
       database: 'test',
     },
   };			
   ```



## 定时任务

使用场景：

1. 定时上报应用状态。
2. 定时从远程接口更新本地缓存。
3. 定时进行文件切割、临时文件删除。

使用：

```js
module.exports = {
  schedule: {
    interval: '1m', // 1 分钟间隔
    type: 'all', // 指定所有的 worker 都需要执行
  },
  async task(ctx) {
    const res = await ctx.curl('http://www.api.com/cache', {
      dataType: 'json',
    });
    ctx.app.cache = res.data;
  },
};
```

## 启动自定义

我们常常需要在应用启动的不同时间进行一些工作，我们可以通过在app.js中返回一个Boot类，在这个类中添加生命周期方法。

- 配置文件即将加载，这是最后动态修改配置的时机（`configWillLoad`）
- 配置文件加载完成（`configDidLoad`）
- 文件加载完成（`didLoad`）
- 插件启动完毕（`willReady`）
- worker 准备就绪（`didReady`）
- 应用启动完成（`serverDidReady`）
- 应用即将关闭（`beforeClose`）

```js
// app.js
class AppBootHook {
  constructor(app) {
    this.app = app;
  }

  configWillLoad() {
    // 此时 config 文件已经被读取并合并，但是还并未生效
    // 这是应用层修改配置的最后时机
    // 注意：此函数只支持同步调用

    // 例如：参数中的密码是加密的，在此处进行解密
    this.app.config.mysql.password = decrypt(this.app.config.mysql.password);
    // 例如：插入一个中间件到框架的 coreMiddleware 之间
    const statusIdx = this.app.config.coreMiddleware.indexOf('status');
    this.app.config.coreMiddleware.splice(statusIdx + 1, 0, 'limit');
  }

  async didLoad() {
    // 所有的配置已经加载完毕
    // 可以用来加载应用自定义的文件，启动自定义的服务

    // 例如：创建自定义应用的示例
    this.app.queue = new Queue(this.app.config.queue);
    await this.app.queue.init();

    // 例如：加载自定义的目录
    this.app.loader.loadToContext(path.join(__dirname, 'app/tasks'), 'tasks', {
      fieldClass: 'tasksClasses',
    });
  }

  async willReady() {
    // 所有的插件都已启动完毕，但是应用整体还未 ready
    // 可以做一些数据初始化等操作，这些操作成功才会启动应用

    // 例如：从数据库加载数据到内存缓存
    this.app.cacheData = await this.app.model.query(QUERY_CACHE_SQL);
  }

  async didReady() {
    // 应用已经启动完毕

    const ctx = await this.app.createAnonymousContext();
    await ctx.service.Biz.request();
  }

  async serverDidReady() {
    // http / https server 已启动，开始接受外部请求
    // 此时可以从 app.server 拿到 server 的实例

    this.app.server.on('timeout', socket => {
      // handle socket timeout
    });
  }
}

module.exports = AppBootHook;
```

## Sequelize(ORM框架)

帮助管理数据库

示例：

1.安装相关依赖

```
npm install --save egg-sequelize mysql2 sequelize-cli
```

2.在config/plugin引入依赖

```
exports.sequelize = {
	enable: true,
	package: 'egg-sequelize'
}
```

3.在config/config.default.js中编写sequelize配置

```js
exports.sequelize = {
	dialect: 'mysql',
	host: '127.0.0,1',
	port: 3306,
	database: 'egg-sequelize-doc-default', //自己定义的数据库名
}
```

4.在mysql中创建自定义数据库

```
mysql> CREATE DATABASE IF NOT EXISTS `egg-sequelize-doc-default`
```

5.通过sequelize-cli创建数据表

​	1.首先在跟目录下创建.sequelizerc文件配置路径

```js
'use strict';

const path = require('path');

module.exports = {
  config: path.join(__dirname, 'database/config.json'), //数据库连接相关
  'migrations-path': path.join(__dirname, 'database/migrations'),//管理数据库变化相关
  'seeders-path': path.join(__dirname, 'database/seeders'),//假数据相关
  'models-path': path.join(__dirname, 'app/model'),//模型关系映射
};
```

​	2.初始化 migrations 和config(等同于config.default.js里的配置)

```
npx sequelize init:config && npx sequelize init:migrations
```

​	3.创建表模板 

```
npx sequelize migration:generate --name=init-users
```

​	4.执行完后会在migration/下生成一个 timestamp-init-users 名字的文件，修改它来初始化表

```js
'use strict';

module.exports = {
  // 在执行数据库升级时调用的函数，创建 users 表
  up: async (queryInterface, Sequelize) => {
    const { INTEGER, DATE, STRING } = Sequelize;
    await queryInterface.createTable('users', {
      id: { type: INTEGER, primaryKey: true, autoIncrement: true },
      name: STRING(30),
      age: INTEGER,
      created_at: DATE,
      updated_at: DATE,
    });
  },
  // 在执行数据库降级时调用的函数，删除 users 表
  down: async queryInterface => {
    await queryInterface.dropTable('users');
  },
};
```

​	5.执行数据库变更

```mysql
npx sequelize db:migrate //升级数据库（可以理解为对表的修改创建等）
```

6.定义model,可以通过app.model或者ctx.model拿到数据表，继而可以对数据表进行操作

```js
//app/model/user.js
'use strict';

module.exports = app => {
  const { STRING, INTEGER, DATE } = app.Sequelize;

  const User = app.model.define('user', {
    id: { type: INTEGER, primaryKey: true, autoIncrement: true },
    name: STRING(30),
    age: INTEGER,
    created_at: DATE,
    updated_at: DATE,
  });

  return User;
};
```

备注：'users'可以写成’user‘。sequelize会帮你加上s

