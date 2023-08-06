# Express简介

- `Express` 是一款在 `Node` 中比较流行的 Web 服务器框架
- 可以基于 `Express` 快速、方便地开发自己的 Web 服务器，并且可以通过工具和中间件来扩展功能
- `Express` 框架的核心就是中间件
- 安装 `express` 框架

```shell
# 安装express框架
npm install express
```

# 基本使用

> **可以通过两种方式使用 `express`**

1. 通过 `express` 脚手架，直接创建应用的骨架

```shell
# 安装脚手架
npm install -g express-generator

# 创建项目
express express-demo

# 安装依赖
npm install

# 启动项目
node bin/www
```

2. 从零搭建自己的 `express` 应用结构

```shell
npm init -y
```

> **使用 `express` 启动一个服务器**

```javascript
const express = require('express');

// 创建express服务器
const app = express();

// 客户端访问的URL：/login(post请求)和/home(get请求)
app.post('/login', (req, res) => {
  res.end('登录成功~');
});
app.get('/home', (req, res) => {
  res.end('Hello World');
});

// 启动服务器，监听8888端口
app.listen(8888, () => {
  console.log('express服务器启动成功');
});
```

# 中间件简介

- `Express` 是一个路由和中间件的Web框架，其本身的功能非常少
- `Express` 应用程序**本质上是一系列中间件函数的调用**

> **中间件：**本质是传给 `express` 的一个回调函数，该回调函数接受三个参数

- 请求对象(request对象)
- 响应对象(response对象)
- next函数(用于执行下一个中间件的函数)

```javascript
// app.post('/login', 回调函数 => 中间件)
app.post('/login', (req, res, next) => {})
```

> **中间件可执行的任务**

- 执行任何代码
- 更改请求(request)和响应(response)对象

```javascript
app.post('/login', (req, res, next) => {
  req.name = 'Jimmy';
});
```

- 结束 **请求 -> 响应** 周期

```javascript
app.post('/login', (req, res, next) => {
  res.end('登录成功');
});
```

- 调用栈中的下一个中间件

```javascript
app.post('/login', (req, res, next) => {
  console.log('第一个中间件')
  next(); // 调用下一个中间件
});

// 使用最基本的中间件
app.use(() => {
  console.log('第二个中间件')
})
```

> **注意：**如果当前中间件没有结束**请求 -> 响应**周期，则必须调用 `next()` 将控制权传递给下个中间件功能，否则请求将被挂起

![1691249908006](images/1691249908006.png)

# 应用级别中间件

- 当 `express` 接收到客户端发送的网络请求时，在所有中间中开始进行匹配
- 当匹配到第一个符合要求的中问件时，那么就会执行这个中间件
- 后续的中间件是否执行取决于上一个中间件是否调用 `next()`

> **`express` 提供了两种方式将中间件注册到应用程序中**，这种注册到 `app` 上的中间件称为应用中间件

- **app.use()：**通过该方法注册的中间件，任何请求方式都可以匹配

- **app.methods()：**其中 `methods` 指的是请求方式，如 `get`，`post` 等

## app.use

- 最普通的中间件

```javascript
app.use((req, res, next) => {
  console.log('first normal middleware');
  next() // 调用下一个中间件
});

app.use((req, res, next) => {
  console.log('second normal middleware');
});
```

- `path` 匹配的中间件(仅对路径作限制)

```javascript
app.use('/home', (req, res, next) => { // 仅匹配/home路径
  console.log('match home middleware');
  res.end('Home Page');
});
```

## app.methods

- `path` 和 `method` 都匹配的中间件

```javascript
// 仅匹配/home路径并且是get请求
app.get('/home', (req, res, next) => {
  console.log('match home get middleware');
  res.end('Home Data');
});

// 仅匹配/users路径并且是post请求
app.post('/users', (req, res, next) => {
  console.log('match users post middleware');
  res.end('Users Data');
});
```

- 一次性注册多个中间件

```javascript
app.get(
  '/home',
  (req, res, next) => {
    console.log('01 match home get middleware');
    next();
  },
  (req, res, next) => {
    console.log('02 match users post middleware');
    next();
  },
  (req, res) => {
    console.log('03 match users post middleware');
    res.end('Home Data');
  }
);
```

# 内置中间件

- `express` 框架提供了一些现成的中间件，可以用于对 `request` 对象进行解析
- `registry` 仓库中也有很多可以辅助开发使用的中间件

## json()

- 使用 `express.json()` 中间件，对客户端传递的 `JSON` 格式数据作解析，并添加到 `req.body` 中

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.post('/login', (req, res) => {
  const { username, password } = req.body;
  console.log(username, password); // 'Jimmy' '123456'
});
```

> **`express.json()` 中间件的原理**

```javascript
app.use((req, res, next) => {
  if (req.headers['content-type'] === 'application/json') {
    req.on('data', (data) => {
      req.body = JSON.parse(data.toString());
    });

    req.on('end', () => {
      next();
    });
  } else {
    next();
  }
});
```

## urlencoded()

- 使用`express.urlencode()` 解析客户端携带的 `urlencoded` 格式的数据，并添加到 `req.body` 中

```javascript
const express = require('express');

const app = express();

app.use(express.urlencoded({ extended: true }));

app.post('/login', (req, res, next) => {
  console.log(req.body); // { username: 'James', password: '123456' }
});

```

> **`extended: true` 的作用：**

- 由于 `urlencode` 默认使用 `node` 内置的 `querystring` 模块解析，但 `querystring` 模块已不推荐使用
- 声明 `extended` 为 `true`时，内部会使用第三方库 `qs`