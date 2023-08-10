# 基本使用

- 安装 `koa` 框架

```shell
npm install koa
```

- 导入并启动一个 `koa` 服务器

```javascript
// 导入koa
const Koa = require('koa');

// 创建app实例
const app = new Koa();

// 启动服务器
app.listen(7777, () => {
  console.log('koa服务器启动成功');
});
```

# Koa中间件

- `koa` 创建的 `app` 对象，注册中间件只能通过 `use` 方法
- `koa` 并**没有提供 `methods` 的方式来注册中间件**，也**没有提供 `path` 中间件来匹配路径**

```javascript
app.use((ctx, next) => {
  ctx.body = 'hello koa'; // 响应数据
});
```

## 参数

- `koa` 的中间件提供了**两个参数：`ctx` / `next`**

> **`koa` 中间件的 `ctx` 参数是一次请求的上下文对象**

- `koa` 并没有像 `express` 一样将 `req` 和 `res` 分开，而是都作为 `ctx` 的属性
- **`ctx.request`：**由 `koa` 封装的请求对象
- **`ctx.req`：**由 `node` 封装的请求对象
- **`ctx.response`：**由 `koa` 封装的响应对象
- **`ctx.res`：**由 `node` 封装的响应对象

```javascript
app.use((ctx, next) => {
  // 请求对象
  console.log(ctx.request); // koa封装的请求对象
  console.log(ctx.req); // node封装的请求对象
  // 响应对象
  console.log(ctx.response); // koa封装的响应对象
  console.log(ctx.req); // node封装的响应对象
});
```

> **`koa` 中间件的 `next` 参数本质是一个dispatch**，类似 `express` 的 `next`

```javascript
app.use((ctx, next) => {
  console.log('normal middleware01');
  next(); // 执行下一个中间件
});

app.use((ctx, next) => {
  console.log('normal middleware02');
});
```

## 区分路径和方法

- 由于 `koa` 并没有提供像 `express` 的 methods 类型的中间件用于区分请求方式
- 并且 `koa` 的中间件只能传递函数，不能传递路径(如 '/login' 这种)

> **`koa` 中有两种方法区分 path 和 method**

- 根据 `request` 自己判断，但这种方法会导致代码很臃肿

```javascript
app.use((ctx, next) => {
  // 区分请求path
  if (ctx.path === '/users') {
    // 区分请求method
    if (ctx.method === 'GET') {
      ctx.body = 'user data';
    } else if (ctx.method === 'POST') {
      ctx.body = 'create user';
    }
  } else if (ctx.path === '/home') {
    ctx.body = 'home data';
  } else if (ctx.path === '/login') {
    ctx.body = 'login data';
  }
});
```

- 使用第三方路由中间件

```shell
# 安装路由
npm i @koa/oruter
```

- 使用路由匹配 path 和 method

```javascript
const userRouter = new KoaRouter({ prefix: '/users' });
// 在路由中注册中间件
userRouter.get('/', (ctx, next) => {
  ctx.body = '用户列表';
});
userRouter.get('/:id', (ctx, next) => {
  ctx.body = '单个用户数据';
});
userRouter.post('/create', (ctx, next) => {
  ctx.body = '创建用户';
});
userRouter.delete('/delete', (ctx, next) => {
  ctx.body = '删除用户';
});
userRouter.patch('/update', (ctx, next) => {
  ctx.body = '更新用户';
});
// 让路由中间件生效
app.use(userRouter.routes());
```

- 当客户端使用了没有定义的方法时，可以使用 `allowedMethods()` 中间件告知客户端
- 如上面接口中没有 `PUT` 方法，假如客户端发送了 `PUT` 请求，不应该返回 `Not Found`

```javascript
// 判断请求方式是否有效，否则返回Method Not Allowed
app.use(userRouter.allowedMethods());
```

# 解析客户端参数

- 解析 `query` 参数，直接通过 `ctx.query` 获取

```javascript
userRouter.get('/', (ctx, next) => {
  ctx.body = '用户的query信息' + JSON.stringify(ctx.query);
});
```

- 解析 `params` 参数，直接通过 `ctx.params` 获取

```javascript
userRouter.get('/:id', (ctx, next) => {
  ctx.body = '用户id为' + ctx.params.id;
});
```

- 使用第三方库 `koa-bodyparser` 解析 `json` 格式参数，然后通过 `ctx.request.body` 获取

```shell
# 安装koa-bodyparser
npm i koa-bodyparser
```

```javascript
const bodyParser = require('koa-bodyparser');
app.use(bodyParser());

userRouter.post('/json', (ctx, next) => {
  const { name, age } = ctx.request.body;
  ctx.body = `创建用户的信息：用户名：${name},年龄：${age}`;
});
```

- 解析 `urlencoded` 格式参数，和 `json` 一致

```javascript
const bodyParser = require('koa-bodyparser');
app.use(bodyParser());

userRouter.post('/urlencoded', (ctx, next) => {
  const { name, age } = ctx.request.body;
  ctx.body = `创建用户的信息：用户名：${name},年龄：${age}`;
});
```

- 使用第三方库 `@koa/multer` 解析 `form-data` 格式参数

```shell
npm i @koa/multer multer
```

```javascript
const multer = require('@koa/multer');
const formParser = multer();

userRouter.post('/formdata', formParser.any(), (ctx, next) => {
  const { name, age } = ctx.request.body;
  ctx.body = `创建用户的信息：用户名：${name},年龄：${age}`;
});
```

# 文件上传

- 单个文件上传，使用 `@koa/multer` 进行解析，使用方式和 `express` 框架的 `multer` 一致

```javascript
const multer = require('@koa/multer');

const upload = multer({
  storage: multer.diskStorage({
    destination(req, file, cb) { // 文件的存储目录
      cb(null, './uploads');
    },
    filename(req, file, cb) { // 对文件重命名
      cb(null, file.originalname);
    }
  })
});

uploadRouter.post('/avatar', upload.single('avatar'), (ctx, next) => {
  console.log(ctx.request.file); // 文件信息
  ctx.body = '文件上传成功~';
});
```

- 多个文件上传

```javascript
uploadRouter.post('/photos', upload.array('photos'), (ctx, next) => {
  console.log(ctx.request.files); // 多个文件信息
  ctx.body = '文件上传成功~';
});
```

# 静态资源部署

- `koa` 并没有内置部署相关的功能，所以需要使用第三方库 `koa-static`

```shell
npm i koa-static
```

- 部署过程和 `express` 框架过程一致

```javascript
const Koa = require('koa');
const static = require('koa-static');

const app = new Koa();

app.use(static('./uploads'));

app.listen(8888, () => {
  console.log('koa服务器启动成功');
});
```

# 数据的响应

- `koa` 框架的响应结果是通过 `ctx.body` 进行输出

> **`body` 将响应体设置为以下之一**

- 字符串数据

```javascript
userRouter.get('/', (ctx, next) => {
  ctx.body = 'Hello World';
});
```

- Buffer数据

```javascript
userRouter.get('/', (ctx, next) => {
  ctx.body = Buffer.from('hello koa');
});
```

- 流数据 

```javascript
const fs = require('fs');

userRouter.get('/', (ctx, next) => {
  const readStream = fs.createReadStream('./uploads/lcw.png');
  ctx.type = 'image/png'; // 设置流的类型为image/png
  ctx.body = readStream;
});
```

- 对象或数组

```javascript
userRouter.get('/', (ctx, next) => {
  ctx.body = {
    code: 200,
    message: 'success',
    data: []
  };
});
```

- 不输出任何内容，设置为 `null` ，此时 http 状态码自动设置为 204

```javascript
userRouter.get('/', (ctx, next) => {
  ctx.body = null;
});
```

- 如果 `response.status` 尚未设置，`Koa` 会自动将状态设置为 200 或 204

# 错误处理

- `koa` 的和 `express` 错误处理逻辑大致相同，也是自定义一套状态码，告知客户端某个状态码对应的错误

> **处理方式上与 `express` 的差异**

- `express` 是通过调用 `next()` 函数，并携带对应的错误码，交由错误处理中间件处理
- `koa` 的 `next()` 函数不能传递参数，而是通过 `ctx` 对象上的 `app` 属性处理，`app` 就是当前应用，并且 `app` 本质上是 `EventEmitter` 对象，可以发送和监听事件

> **错误处理方式如下**

- 通过 `app.emit()` 发送一个 `error` 事件，并传递错误码和当前 `ctx` 对象

```javascript
userRouter.post('/', (ctx, next) => {
  const { name, password } = ctx.request.body;
  if (!name || !password) {
    // 错误处理
    ctx.app.emit('error', -1001, ctx);
  } else if (name !== 'admin' || password !== '123456') {
    // 错误处理
    ctx.app.emit('error', -1002, ctx);
  } else {
    // 请求成功
    ctx.body = '登录成功';
  }
});
```

- 通过 `app.on()` 监听 `error` 事件，并拿到传递的错误码和 `ctx`

```javascript
app.on('error', (errCode, ctx) => {
  const message = new Map([
    [-1001, '用户名或密码不能为空'],
    [-1002, '用户名或密码错误']
  ]).get(errCode);

  ctx.body = {
    message,
    code: errCode
  };
});
```