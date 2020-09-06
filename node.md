# 1.常用API

- process.pwd()和___dirname 
- - process.pwd是node运行时所在路径
  - __dirname是文件所在路径
  - __filename是文件所在路径+文件名

- path.resolve和path.join

- - path.resolve是从右往左解析参数生成一个路径，如果这个路径还不到绝对路径，那么当前工作目录会继续被拼接 
  - path.join 拼接路径

- path.dirname(''):返回路径的父级路径

- path.basename('')，返回路径最后一个文件或文件夹名

- path.sep，返回文件分割符 ’/‘或’\‘

- path.parse()

- fs.readFileSync(url, utf8);同步读取文件

- process.agrv.splice(2)获取传入node的参数

- process.exit();结束程序

- buffer.from 复制并转换成二进制数组 

- asset.equal();断言判断是否内容一样

- fs.createReadStream('index.txt'); //读取大文件

- fs.createWriteStream('out.txt');//创建大文件

# 3.node模块介绍

## 3.1Buffer模块

​	Buffer是一个像Array的对象，主要用来操作字节。在应用中我们常常操作字符串，在网络传输中，经常以二级制传输。

​	Buffer所占的内存不是通过V8分配的，属于堆外内存。它是在C++层面实现内存申请的。

​	Buffer挂载在全局上，无需require引入。

```js
var str = 'ihgifndakdnd';
var buf = new Buffer(str, 'utf-8');
console.log(buf);
// <buffer e6 e5 e4 12>
```

### Buffer的转换

​	buffer和字符串直接可以相互转换。目前支持的字符串编码格式如下

- ASCII
- UTF-8
- UTF-16LE/UCS-2
- Base64
- Binary
- Hex

**字符串转buffer**

```js
new Buffer(str, [encoding]); //已废弃
Buffer.from(str, [encoding]); //新方案
```

备注：encoding默认为utf8

**buffer转字符串**

```js
buffer.toString([encoding], [start], [end]);
```



### Buffer的拼接

Buffer在拼接中文类型可能会出现问题，下面介绍它的错误使用以及正确使用

**错误**：直接使用+号拼接

```js
const fs = require('fs');
const rs = fs.createReadStream('test.md');
let data = '';

rs.on('data', function(chunk) {
    data+= chunk;
});
rs.on('end', function() {
	console.log(data);
});
```

**正确**：使用数组来接收所有Buffer片段，并记录片段总长度，然后调用Buffer.concat方法合成新的Buffer对象

```js
let chunks = [];
let size = 0;
rs.on('data', function(chunk) {
	chunks.push(chunk);
	size+= chunk.length;
});
rs.on('end', function() {
	const buf = Buffer.concat(chunks, size);
	const str = buf.toString('uft8');
	console.log(str)
})
```

## 3.2HTTP模块

常用API

http.createServer([options]，[, requestListener]);//创建 http服务

request.end([data,[, encoding]],[, callback]); 返回给客户端数据

response.setHeader(name, value);//设置相应头部

response.writeHead(statusCode,[,message],[,headers]); //相同头部优先级高于setHeader设置的头部

response.getHeader(name);//获取还未返回给客户端通过setHeader设置的响应头

## 3.3URL模块

用来帮助我们方便快捷的处理url。

以前版本是通过node的url包来处理，现在推荐使用全局变量URL来处理，下面只介绍新方法

**基本使用**

```js
const myUrl = new URL('http://example.com/p/a/t/h?query=string#hash1');
/**URL {
  href: 'http://example.com/p/a/t/h?query=string#hash1',
  origin: 'http://example.com',
  protocol: 'http:',
  username: '',
  password: '',
  host: 'example.com',
  hostname: 'example.com',
  port: '',
  pathname: '/p/a/t/h',
  search: '?query=string',
  searchParams: URLSearchParams { 'query' => 'string' }, 
  hash: '#hash1'
}*/
myURL.searchParams.get('query'); // string
myURL.searchParams.append('abc', 'xyz'); 
myURL.searchParams.delete('abc');
myURL.searchParams.set('a', 'b'); 

```

### URLSearchParams

用来处理url参数的全局变量,上面的searchParams就是URLSearchParams的一个实例。我们也可以单独使用 new URLSearchParams($1)来创建一个方便操作请求参数的对象;其中$1可以为以下四种情况

1. 空： Instantiate a new empty `URLSearchParams` object

2. string 'user=abc&query=xyz' 或者 '?user=abc&query=xyz'。?可省略

3. obj 

   ```
   const params = new URLSearchParams({
     user: 'abc',
     query: ['first', 'second']
   });
   ```

4. iterable 支持元素为键值对的iterator对象

   ```
   function* getQueryPairs() {
     yield ['user', 'abc'];
     yield ['query', 'first'];
     yield ['query', 'second'];
   }
   params = new URLSearchParams(getQueryPairs());
   ```

**URLSearchParams实例支持的API如下**

```js
append(name, value)
set(name, value)
get(name)
getAll(name)
delete(name)
has(name)
entries() //返回一个ES6迭代器对象
forEach(fn[, thisArg])
keys()
toString()
sort()
[Symbol.iterator]()
```

## 3.4Fs模块

文件操作相关

- fs下的stats类主要用来查看文件信息

- fs.renameSync(oldPath, newPath)：重命名
- const  w = fs.createWriteStream(targetFile, { flags: 'a'}) : 创建写入流
- - flag: 设置为‘a’就是追加而不是替换原本文件里的内容
- const r = fs.createReadStream(currentFile): 创建写出流
  - r.pipe(w, { end: false })：自动将写出流传输到写入流
    - end: 写出结束时是否关闭写入流







# 4.如何用node构建web应用 

## **构建一个web应用，我们常常需要应对以下需求**

1. 创建HTTP服务
2. 路由
3. URL路径解析和**GET**请求处理
4. **Post**请求处理
5. 文件上传处理
6. 文件下载处理
7. 重定向 
8. Cookie解析
9. 缓存

## 原生API实现

备注：下面的实现一些重复代码将会被省略

### 1.创建HTTP服务

```js
const http = require('http');
const server = http.createServer(function(req, res) {
    // ...
});
server.listen(3005);
```

### 2.路由

包括

1. 重定向
2. 
3. 静态资源托管
4. api处理

```js
//这里假设我们的静态资源目录为当前目录下的resource目录下面
//未登录用户进行重定向
if (!isLogin(req.headers.cookie) && !req.url.startWith('/login')) {
        res.writeHead(302, {
            'Location': 'login.html'
        });
        res.end();    
}
if (req.url == '/') {
    	//读取文件响应给客户端
        fs.readFile(path.join(__dirname, 'resource', 'html' ,'index.html'),(err, data)=>{
            if(err){
                throw err;//如果读取失败，抛出异常
            }else{
                res.end(data);//如果读成功，响应给客户端
            }
        });
 } else if (req.url.startWith('/resource')){//只要路径以/resource开头，直接拼接返回
        //读取文件响应给客户端
        fs.readFile(path.join(__dirname, req.url),(err,data)=>{
            if(err){
                throw err;//如果读取失败，抛出异常
            }else{
                res.end(data);//如果读成功，响应给客户端
            }
        });
 } else if (req.url.startWith('/api')) { //api请求通过router类去处理,主要通过url和method分发
     	router.hand(req, res);  //这里只写思路，具体就不实现了
 }
```

### 3.URL路径解析和**GET**请求处理

```js
// 已过时的写法
//const url = require('url');
//url.parse(urlString[, parseQueryString[, slashesDenoteHost]]);

// 新用法 
let myUrl = new URL(req.url || 'http://example.com/p/a/t/h?query=string#hash1');
/**URL {
  href: 'http://example.com/p/a/t/h?query=string#hash1',
  origin: 'http://example.com',
  protocol: 'http:',
  username: '',
  password: '',
  host: 'example.com',
  hostname: 'example.com',
  port: '',
  pathname: '/p/a/t/h',
  search: '?query=string',
  searchParams: URLSearchParams { 'query' => 'string' }, 
  hash: '#hash1'
}*/
myURL.searchParams.get('query'); //获取约定好的参数
```

### 4.post请求处理

```js
// post请求
var querystring = require('querystring');
		//1.给req对象注册一个data事件：表示开始接收post请求参数
        //客户端没发送一次数据该方法都会执行一次，回调函数的参数就是本次接收到的数据（数据流）
        //接受的次数不固定：取决于数据的大小和你的宽带的网速
        let postData = "";
        req.on('data', function (chuck) {
            postData += chuck;
        });

        //2.给req对象注册一个end事件，表示本次post数据发送完毕
        req.on('end', function () {
            //当客户端本次post请求数据发送完毕之后，会执行这个函数
            console.log(postData);
            //3.解析post参数得到参数对象
            //使用querystring模块来解析post请求的参数
            var postObjc =  querystring.parse(postData);
            console.log(postObjc);
            //响应客户端
            res.end(JSON.stringify(postObjc));
        });
```

### 5.Cookie设置

```js
let cookies = req.headers.cookie;
//...
res.setHeader('Set-Cookie', str);
```

### **6.缓存**

配置GET请求的强缓存和协商缓存

```js
//协商缓存
function handle(req, res) { //兼容http1.0
	fs.stat(filename, function(err,stat) {
        const lastModified = stat.mtime.toUTCSring();
        if (lastModified === req.headers['if-modified-since']) {
            res.writeHead(304, "Not Modified");
            res.end()
        } else {
            fs.readFile(filename, function(err, file) {
                res.setHeader('Last-Modified', lastModified);
                res.writeHead(200, 'ok');
                res.end(file);
            })
        }
    })
}
function handle2(req, res) {//处理http1.1
	fs.readFile(filename, funciton(err, file) {
    	const hash = getHash(file); //getHash为自定义的生成规则
    	const noneMatch = req.headers['if-none-match'];
    	if (hash === noneMatch) {
            res.writeHead(304, 'Not Modified');
            res.end();
        } else {
            res.setHeader('ETag', hash);
            res.writeHead(200, 'ok');
            res.end(file);
        }
    })
}
```

```js
//强缓存
function hander(req, res) { // 处理http1.0 expires
    fs.readFile(filename, function(err, file) {
        let expires = new Date();
        expires.setTime(expires.getTime() + 365 * 24 * 60 * 60 * 100 );
        res.setHeader('Expires', expires.toUTCString()); 
        res.writeHead(200, 'ok');
        res.end(file);
    })
}
function hander(req, res) { //处理1.1 Cache-Control
	fs.readFile(filename, function(err, file) {
		res.setHeader('Cache-Control', 'max-age=' + 365 * 24 * 60 * 60 * 100);
        res.writeHead(200, 'ok');
        res.end(file);
    })
}
```

### 7.文件上传处理 

https://segmentfault.com/a/1190000015558975

### 8.文件下载

```js
const http = require('http');
const fs = require('fs');
const path = require('path');

const filename = path.join(__dirname, 'download.js');
const server = http.createServer(function(req, res) {  
    fs.stat(filename, function(err, stat) {
        let stream = fs.createReadStream(filename);
        res.setHeader('Content-Type', 'application/javascript');
        res.setHeader('Content-Length', stat.size);
        res.setHeader('Content-Disposition', 'attachment;filename="' + path.basename(filename) + '"');
        res.writeHead(200);
        stream.pipe(res);
    }) 
});
server.listen(3005);
```



# 5.调试问题总结

## 5.1package.json因为本地包问题

报错：package.json依赖本地包报 it does not contain a package.json file ；

解决：路径换为绝对路径

```json
"egg-tracer": "file:D:/开发/代码/koa/egg-tracer"
```

# 6.node实用的包

## globby，返回指定条件下所有文件的相对路径

```js
const files = globby.sync('**/*.js', {//查找path路径下面的所有js文件的相对路径
            cwd: path
        });
```

## vinyl-fs,常用于复制文件，将文件信息通过对象来描述，可改变对象属性来改变文件内容。

```js
var map = require('map-stream');
var vfs = require('vinyl-fs');
 
var log = function(file, cb) {
  console.log(file.path);
  cb(null, file);
};
 
vfs.src(['./js/**/*.js', '!./js/vendor/*.js']) //指定文件
  .pipe(map(log))
  .pipe(vfs.dest('./output'));//模板目录
```

## map-stream,转换流中数据并返回

```js
var map = require('map-stream')

map(function (data, callback) {
  //transform data
  // ...
  callback(null, data)
})
```

## concat-stream ,用来获取流中的内容，把所有内容拼接起来一次返回

## lodash.template,字符串替换函数

```js
lodash.template('<%= name %>')({name: 123})//name会被123替换
```

## yargs命令行工具，同commander，inquirer.js（推荐这个，简单好用）

```js
const {argv} = require('yargs')

if (argv.ships > 3 && argv.distance < 53.5) {
  console.log('Plunder more riffiwobbles!')
} else {
  console.log('Retreat from the xupptumblers!')
}
// 运行
// ./plunder.js --ships=4 --distance=22
//  Plunder more riffiwobbles!
```

## chalk,自定义终端样式库

```js
chalk.red('the word color is red');
```

## tildify，转换为相对路径

```js
let newPath = tildify(path);
```

## chokidar,监听文件变化

```js
chokidar.watch('./app').on((event, path) => {
	//...
})
```

## cluster-reload 重启进程

```js
const reload = require('cluster-reload');
//when change, exec reload() 
```

## cfork 多进程运行

```js
cfork({
	exec: path.resolve(__dirname, 'index.js'),
	count: require('os').cpus().length
}).on('fork', worker => {
	console.log('[%s] [worker:%d] new worker start', Date(), worker.process.pid)
}).on('disconnect', () => {
	//...todo
}).on('exit', () => {
	//...todo
})
```

# socket.io-client 封装了客户端websocket

```js
import io from 'socket.io-client';
var socket = io('http://localhost');
  socket.on('connect', function(){});
  socket.on('event', function(data){});
  socket.on('disconnect', function(){});
```

链接：https://github.com/socketio/socket.io-client#readme

## zlib: gizp压缩

```js
const zlib = require('zlib');
const stream = zlib.createGzip();
stream.end(JSON.stringify(ctx.body));
ctx.set('Content-Encoding', 'gizp');
```

## [factory-girl](https://github.com/simonexmachina/factory-girl) :打数据

## cross-env:设置环境变量，不用担心跨平台问题

## node-heapdump 查找内存泄漏问题

## Crawler 爬虫

# 7.部分node DEMO

## 创建简单HTTP请求

```js
const http = require('http');
const server = http.createServer(function(req, res) {
    const credentials = auth(req);
    if (!credentials) {
        res.statusCode = 401;//设置状态码
        res.setHeader('WWW-Authenticate', 'Basic realm="example"');//设置响应头
        res.end('Access denied');//输出响应体
    } else {
        res.end('Access granted');
    }
});
server.listen(3004);
```

**注意：**auth是用来鉴别登录用户是否允许登录，设置WWW-Authenticate响应头能让浏览器弹出框来输入账号密码；  

## 读写大文件

```js
const reader = fs.createReadStream('index.txt');
const writer = fs.createWriteStream('out.txt');

reader.on('data', function(chunk) {
	writer.write(chunk);
});
reader.on('end', function() {
    writer.end();
});

//由于读写模式固定，可以使用下面更简洁的方式
// reader.pipe(writer);
```
