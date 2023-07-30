# Stream读写操作

> **`stream` 是什么？**

- 从一个文件中读取数据时，文件的二进制数据会源源不断被读取到程序中，而这种一连串的字节，就是**程序中的流(stream)**
- 流是**连续字节的一种表现形式和抽象概念**，同时是**可读可写**的

> **既然可以通过 `readFile` 或 `writeFile` 读写文件，为何需要流？**

- **无法控制细节：**如开始读的位置、结束位置、一次性读取多少字节等
- **读取不可控：**如当读到某个位置时需要暂停读取，在某个时刻又恢复继续读取
- **文件过大：**如视频文件一次性全部读取并不合适

> **所有的流都是 `EventEmitter` 的实例，`Node` 中的四种基本流类型：**

- **Writable：**可以向其写入数据的流，如 `fs.createWriteStream()`
- **Readable：**可以从中读取数据的流，如 `fs.createReadStream()`
- **Duplex：**同时为 `Readable` 和 `Writable`，如 `net.Socket`
- **Transform：**`Duplex` 可在写入和读取数据时修改或转换数据的流，如 `zlib.createDeflate()`

## Readable可读流

- 通过 `fs.createReadStream()` 开启可读流

```javascript
const readStream = fs.createReadStream('./aaa.txt', {
  start: 3, // 流的起始位置
  end: 15, // 流的结束位置
  highWaterMark: 4 // 每次读取的字节数，默认是64kb
});
```

- 由于流是 `EventEmitter` 实例，可以监听 `data` 事件，流读取完成后会触发

```javascript
// 监听流读取完成的事件
readStream.on('data', (chunk) => {
  console.log('接收到的数据', chunk.toString());
});
```

- 流的暂停

```javascript
readStream.pause();
```

- 流的继续读取

```javascript
readStream.resume();
```

- 流的其他事件

```javascript
// 通过流打开文件后触发的事件
readStream.on('open', (fd) => {
  console.log('文件打开成功', fd);
});

// 数据读取完毕触发的事件
readStream.on('end', () => {
  console.log('数据读取完毕');
});

// 通过流关闭文件后触发的事件
readStream.on('close', () => {
  console.log('文件关闭成功');
});
```

## Writable可写流

- 通过 `fs.createWriteStream()` 开启可写流

```javascript
const fs = require('fs');

const writableStream = fs.createWriteStream('./bbb.txt', {
  flags: 'a+'
});
```

- 通过其 `write` 方法，向文件中写入内容

```javascript
writableStream.write('Hello Jimmy', (err) => {
  if (err) return console.log('写入失败', err);
  console.log('写入成功');
});
```

- 内容写入完成后，由于写入流不会自动关闭，则需要手动调用其 `close` 方法关闭文件

```javascript
writableStream.close();
```

- 也可以通过其 `end` 方法，将最后的内容写入文件中，并关闭文件

```javascript
writableStream.end('finish content');
```

- 监听流的事件

```javascript
// 监听文件被关闭事件
writableStream.on('close', () => {
  console.log('文件关闭成功');
});

// 监听文件被打开事件
writableStream.on('open', (fd) => {
  console.log('文件描述符：', fd);
});

// 监听内容写入完成事件
writableStream.on('finish', () => {
  console.log('内容写入完成');
});
```

## pipe方法

- 通过 `pipe` 方法，将可读流的内容之间放入到可写流

```javascript
const fs = require('fs');

// 创建可读流
const readStream = fs.createReadStream('./ccc.txt');
// 创建可写流
const writeStream = fs.createWriteStream('./ccc_copy.txt');
// 在两个流之间建立一个管道,将可写流的数据放入到可写流
readStream.pipe(writeStream);
```

# http模块

- 在 `Node` 中，提供 `web` 服务器的资源返回给浏览器，主要是通过 `http` 模块
- 利用 `http.createServer()` 方法，创建服务器对象，其底层是直接 `new Server` 对象

> **创建Server时传入的回调携带两个参数**
>
> - **req：**请求对象，包含客户端请求相关的信息
> - **res：**响应对象，包含发送给客户端的信息

```javascript
const http = require('http');

// 创建http服务器
const server = http.createServer((req, res) => {
  // request对象用于获取客户端的请求信息
  console.log(req);
  // response对象用于给客户端响应结果
  res.end('Hello World');
});
```

- 通过 `server.listen()` 方法开启服务器，并监听端口号

```javascript
// 开启服务器，监听8888端口
server.listen(8888, () => {
  console.log('服务器启动成功');
});
```

![1690737921793](images/1690737921793.png)

## request对象

> **向服务器发送请求时携带的信息：**

- **请求的 `URL`：**服务器需要根据不同的URL进行不同的处理
- **请求的方式：**如GET、POST请求传入的参数和处理的方式是不同的
- **请求的 `headers`：**如客户端信息、接受数据的格式、支持的编码格式等

```javascript
const server = http.createServer((req, res) => {
  // req对象中包含的信息
  console.log('url:', req.url); // url
  console.log('method:', req.method); // method
  console.log('headers:', req.headers); // headers
  res.end('Hello World');
});
```

![1690742131835](images/1690742131835.png)

### url和method处理

- 服务器需要根据不同的请求地址，做出不同响应

```javascript
const server = http.createServer((req, res) => {
  res.end(distinguishUrl(req.url));
});

const distinguishUrl = (url) => {
  return new Map([
    ['/login', '登录成功~'],
    ['/products', '商品列表~'],
    ['/lyric', '歌词数据~']
  ]).get(url);
};
```

- 服务器还需要判断支不支持当前的请求方式

```javascript
const server = http.createServer((req, res) => {
  const { method, url } = req;
  if (url === '/login') {
    method === 'POST' ? res.end('登录成功~') : res.end('不支持该请求方法~');
    return;
  }
  res.end('Hello World');
});
```

### query参数













