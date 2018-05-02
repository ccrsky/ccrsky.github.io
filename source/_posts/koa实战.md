---
title: koa实战
date: 2018-04-19 19:24:36
category:
    - node

tags:
    - js
    - node
---



### koa实战-快速打造一个CMS系统

### koa介绍

基于Node.js平台的下一代web开发框架

由 Express 原班人马打造的 koa，致力于成为一个更小、更富有表现力、更健壮的 web 开发框架。 使用 koa 编写 web 应用，通过组合不同的 generator，可以免除重复繁琐的回调函数嵌套， 并极大地提升常用错误处理效率。Koa 不在内核中打包任何中间件，它仅仅提供了一套优雅的函数库， 使得编写 Web 应用变得得心应手。

>node.js环境 版本v7.6以上,koa2

### 快速开始

1.koa安装

```sh
npm install koa
```

2.编写app.js

```js
const Koa = require('koa')
const app = new Koa()

app.use( async ( ctx ) => {
  ctx.body = 'hello koa2'
})

app.listen(3000)
```

3.启动web服务

```js
node app.js
```


#### koa中间件
koa对网络请求采用了中间件的形式处理,中间件可以介入请求和相应的处理,是一个轻量级的模块,每个中间负责完成某个特定的功能。中间件的通过next函数联系,执行next()后会将控制权交给下一个中间件,如果没有有中间件没有执行next后将会沿路折返,将控制权交换给前一个中间件

![中间件](https://raw.githubusercontent.com/fengmk2/koa-guide/master/onion.png)


##### 创建中间件

新建 md.js

```js
module.exports = function () {
    return async function(ctx, next) {
      const startDate = new Date();
      next();
      console.log(`method: ${ctx.method} code: ${ctx.status} time:${new Date() -startDate}ms`);
    }
}
```

##### 使用中间件

```js
const Koa = require('koa') 
const logger = require('./md')
const app = new Koa()

app.use(logger())

app.use(( ctx ) => {
    ctx.body = 'hello world!'
})

app.listen(3000)
console.log('the server is starting at port 3000')
```

#### 路由 
##### 原生实现路由

```js
app.use( async ( ctx ) => {
  let url = ctx.request.url

  if(url == 'index.html'){
    fs.readFile('/view/index.html', 'utf8', ( err, data ) => {
      if ( err ) {
        ctx.body ='500';
      } else {
        ctx.body = data;
      }
    })
  }
  
})

// 也可安装模板中间件koa-views，使用其他js模板引擎 
```

> 页面渲染可使用ejs, Jade,Nunjucks等模板引擎。

##### 使用路由中间件
1.安装koa-router

```sh
npm install --save koa-router
```

2.router使用

```js
const Koa = require('koa');
const app = new Koa();
const Router = require('koa-router');

// 定义路由规则
let home = new Router();
home.get('/', async ( ctx )=>{
  ctx.body ='Hello world';
})

// 加载路由
app.use(router.routes()).use(router.allowedMethods());

app.listen(3000, () => {
  console.log('server is starting at port 3000')
})
```

#### cookie/session

##### cookie使用
koa提供了从上下文直接读取、写入cookie的方法

- ctx.cookies.get(name, [options]) 读取上下文请求中的cookie
- ctx.cookies.set(name, value, [options]) 在上下文中写入cookie

koa2 中操作的cookies是使用了npm的[cookies](https://github.com/pillarjs/cookies)模块

```js
// 响应头写入cookie
ctx.cookies.set('sid', 'xmly',{
    domain: 'a.ximalaya.com',  // 写cookie所在的域名
    path: '/index',       // 写cookie所在的路径
    maxAge: 10 * 60 * 1000, // cookie有效时长
    expires: new Date('2018-04-25'),  // cookie失效时间
    httpOnly: false,  // 是否只用于http请求中获取
    overwrite: false  // 是否允许重写
  });
  
// 设置响应内容
ctx.body = 'response body';

```

##### koa-session
koa2原生功能只提供了cookie的操作，但是没有提供session操作,只能自己实现或通过第三方中间件实现，这里我们用到了`koa-session`。

```sh
npm install  koa-session
```
###### 使用cookie存储session数据

```js
// session config
const sessionCfg = {
  // store,
  key: COOKIE_LABEL,    //1&_token
  // maxAge: 86400000,
  path: COOKIE_PATH,
  maxAge:'session',
  overwrite: true, 
  httpOnly: true,
  signed: false, 
  rolling: true,   
  renew: false, 
 };
 
 
 // 使用session中间件
 app.use(session(sessionCfg, app));
 
 
 // 设置session 值 , koa-session提供一个一种hack方法修改cookie的过期时间 _maxAge
 ctx.session.user = _.assign(user,{_maxAge:7 * 24 * 3600 * 1000})
 ctx.session.isLogin = true;
 
 // 清除session数据
 ctx.session = null;

```

###### 使用外部store存储数据

```js
// 外部store必须实现以下3个方法
let memoryStore = {
    storage: {},
    async get(key, maxAge) {
        return this.storage[key]
    },
    async set(key, sess, maxAge) {
        this.storage[key] = sess
    },
    async destroy(key) {
        delete this.storage[key]
    }
}

// 修改session配置
const sessionCfg = {
  store: memoryStore,  // 使用外部store
  key: COOKIE_LABEL,
  // maxAge: 86400000,
  path: COOKIE_PATH,
  maxAge:'session',
  overwrite: true, 
  httpOnly: true,
  signed: false, 
  rolling: true,   
  renew: false, 
 };
```

>如果session数据量很小，可以直接存在内存中,如果session数据量很大，则需要存储介质存放session数据,比如redis,mysql等。


#### 数据库
一个完整的系统当然少不了数据库，这里我们使用mysql。
ORM安装

```sh
npm install sequelize
```

##### 数据库连接配置

```js
const Sequelize = require('sequelize');

// 创建数据库访问实例
const sequelize = new Sequelize('database', 'username', 'password');

// 定义模型
const User = sequelize.define('user', {
  username: Sequelize.STRING,
  description: Sequelize.DATE
});

// 自动建表
sequelize.sync({ force: true });

// 数据持久化
User.create({ username: 'jason.chen', description: '全栈开发' }).then(function(user) {
  // user 是一个持久化实例
})
```

##### Model操作

```js
// 新增
User.create({ username: 'jason.chen', description: '全栈开发' })

// 删除
User.destroy({where:{id:1}})

// 更新
User.update({description:'全栈开发'},{where:{id:1}})

// 查询
User.findOne({where:{id:1}});

```

#### 开发调试
node调试方式有三种

- VSCode调试模块
- Chrome 调试
- 打印日志调试(debug模块)

这里使用Chrome调试，依赖环境如下

- node环境8.x+
- chrome 60+

```sh
node --inspect app.js
```
> debug模块可以根据debug环境变量，控制不同命名空间的日志输出，开发过程中可使用nodemon根据文件修改自动重启服务。


### 项目框架搭建

#### 开发环境

- git
- node (v8.11.1)
- mysql (v5.6+)
- redis


#### 目录结构

```js
.
├── LICENSE
├── README.md
├── api.md  // api接口文档
├── ecosystem.config.js  // pm2配置
├── logs  // 应用日志
├── package-lock.json
├── package.json
├── src
│   ├── app.js  // 应用入口文件
│   ├── config
│   │   ├── app.js
│   │   ├── db.js   // db配置文件
│   │   ├── middleware.js  // 中间件
│   │   ├── redis.js    // redis配置文件
│   │   └── session.js  
│   ├── controllers
│   │   ├── sys.js
│   │   └── user.js // 用户模块控制器
│   ├── models
│   │   ├── post.js
│   │   └── user.js // 用户模型
│   ├── routers
│   │   ├── sys.js
│   │   └── user.js // 用户模型
│   ├── run.js  // 系统初始化脚本(初始化数据+mock数据)
│   ├── test        // 测试demo
│   │   ├── db.js
│   │   └── test.js
│   └── utils
│       ├── db.js
│       ├── index.js
│       ├── log.js
│       ├── mail.js
│       ├── redis.js
│       └── store.js
└── package.json

```

#### 功能模块

- 多级路由
- 应用日志
- 参数校验
- 权限控制
- 负责均衡
- 邮件通知
- 钉钉通知

##### 多级路由

全局路由模块

```js
let router = new Router();

const user = require('./user')
const post = require('./post')

router.use('/user', user.routes(), user.allowedMethods())
router.use('/post', post.routes(), post.allowedMethods())
```

具体路由模块

```js
const routers = router
    .post('/login', validate(user.v.login), user.login)
    .post('/logout', user.logout)
```


##### 日志

###### 应用日志
应用日志这里使用了`log4js`

log4js配置

```js
module.exports = {
  "appenders": {
    "access": {
      "type": "dateFile",
      "filename": "/var/log/myApp/access.log",
      "pattern": "-yyyy-MM-dd",
      "category": "http"
    },
    "app": {
      "type": "file",
      "filename": "/var/log/myApp/app.log",
      "maxLogSize": 10485760,
      "backups": 3
    }
  },
  "pm2": true,
  "categories": {
    "default": { "appenders": [ "app"], "level": "DEBUG" },
    "http": { "appenders": [ "access"], "level": "DEBUG" }
  }
}
```
实例化logger

```js
const log4js = require('log4js');

let config = require('../config/log');
log4js.configure(config);

module.exports = {
    appLogger:log4js.getLogger(),
    httpLogger:log4js.getLogger('http')
};
```

日志记录

```js
const { appLogger , httpLogger} = require('./utils/log');

app.on('error', function(err, ctx) {
      appLogger.error(err)
});

```


###### 访问日志
access日志这里使用了`koa-logger`

```js
const logger = require('koa-logger')

app.use(logger((str, args) => {
  if(useFileLogger()){
    httpLogger.info(`${args[1]} ${args[2]} ${args[3] || ''} ${args[4] || ''} ${args[5] || ''}`)
  }else{
    console.log(str)
  }
}))
```

##### 参数校验
web前端输入内容都是不可信的，后端必须校验请求参数，这里用到了`joi`。

![joi](https://raw.github.com/hapijs/joi/master/images/joi.png)
Object schema description language and validator for JavaScript objects.

```js
const Joi = require('joi');
 
const schema = Joi.object().keys({
    username: Joi.string().alphanum().min(3).max(30).required(),
    password: Joi.string().regex(/^[a-zA-Z0-9]{3,30}$/),
    birthyear: Joi.number().integer().min(1900).max(2013),
    email: Joi.string().email()
});
 
// Return result.
const result = Joi.validate({ username: 'abc', birthyear: 1994 }, schema);
// result.error === null -> valid
 
// You can also pass a callback which will be called synchronously with the validation result.
Joi.validate({ username: 'abc', birthyear: 1994 }, schema, function (err, value) { });  // err === null -> valid
```


##### 权限控制

鉴权中间件

```js
let auth = (authority)=>{
    return async(ctx, next) => {
        if(!ctx.session || (ctx.session && !ctx.session.isLogin)){
            const error = new Error('请先登录');
            error.status = 401;
            throw error;     
        }

        if(authority == 'admin' && !ctx.session.isAdmin){
            const error = new Error('权限不足');
            error.status = 403;
            throw error; 
        }
        
        await next();
    };
}
```
鉴权中间件使用

```js
const routers = router
    .post('/login', validate(user.v.login), user.login)
    .get('/getCurrent', auth(), user.getCurrent)
    .get('/getUser', auth('admin'),validate(user.v.getUserById), user.getUserById)
```

##### 负责均衡

###### pm2多进程
![负载均衡](http://i.imgur.com/kTAowsL.png)
通过pm2进程管理工具管理应用,实现多进程负载

```js
module.exports = {
    apps: [{
        name: 'cms-enterprise-helper',
        script: './src/app.js',
        instances : 1,  // 'max',
        // instances : 1,
        exec_mode: 'cluster',
        merge_logs : true,
        log_file:   '/var/log/myApp/app.log',
        error_file: '/var/log/myApp/app-err.log',
        out_file :  '/var/log/myApp/app-out.log',
        log_date_format : 'YYYY-MM-DD HH:mm Z'
    }]
}
```

###### nginx负载均衡

nginx部分配置

```js
#设定负载均衡的服务器列表
upstream mysvr {
    server 127.0.0.1:3000 weight=1;
    server 192.168.3.0.60 weight=1;
    # server backend1.example.com       weight=5;
}

server {
    listen       80;
    server_name  cc.ximalaya.com  ;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size 10m;    
    client_body_buffer_size 128k;  
    proxy_connect_timeout 90;  
    proxy_send_timeout 90;        
    proxy_read_timeout 90;         
    proxy_buffer_size 4k;             
    proxy_buffers 4 32k;              
    proxy_busy_buffers_size 64k;    
    proxy_temp_file_write_size 64k; 
    
    location /api/ {
        proxy_pass  http://mysvr/;  
    }

    
}
```

##### 邮件通知
![nodemailer](https://raw.githubusercontent.com/nodemailer/nodemailer/master/assets/nm_logo_200x136.png)

```js
let nodemailer  = require('nodemailer');

let {
     host,
    secure, 
    username,
    password
} = require('../config/mail');

let transporter = nodemailer.createTransport({
    host: host,
    secure: secure, 
    auth: {
        user: username, 
        pass: password 
    }
});

module.exports = function sentMail(mail) {
    return transporter.sendMail(Object.assign(mail,{from: '"系统管理员" <rsky163@163.com>'}));
}


// 发送邮件
let mail = {
    from: '"系统管理员" <rsky163@163.com>', // sender address
    to: 'jason.chen@ximalaya.com', // list of receivers
    subject: 'Hello ✔', // Subject line
    text: 'Hello world -- text?', // plain text body
    html: '<b>Hello world == html</b>', // html body
};

sendMail(mail)
```

##### 钉钉通知

通过`request`封装钉钉机器人接口请求

```js
const ddUtil = {
    send(url, obj){
        return request({
            method: 'POST',
            uri: url,
            json:true,
            headers: [{
                name: 'content-type',
                value: 'application/json'
            }],
            body: obj
        })
    }
}

ddUtil.send(url_with_token,{
    "msgtype": "text", 
    "text": {
        "content": "Hello ximalaya !"
    }
})

```
>钉钉机器人，更多用法参见 https://open-doc.dingtalk.com/docs/doc.htm?spm=a219a.7629140.0.0.karFPe&treeId=257&articleId=105735&docType=1

#### 上线部署

部署构图如下：
![项目部署图](http://ouonaooa5.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-08%2020.46.33.png)

Node服务进程管理工具:

- [pm2 推荐](http://pm2.keymetrics.io/)
- [forever](https://github.com/foreverjs/forever)