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





