# 使用中间件

从本质上来说，一个 Express 应用就是在调用各种中间件。

中间件是一个可访问请求对象（`req`）和响应对象（`res`）的函数，在 Express 应用的请求-响应循环里，下一个内联的中间件通常用变量 `next` 表示。中间件的功能包括：

- 执行任何代码。
- 修改请求和响应对象。
- 终结请求-响应循环。
- 调用堆栈中的下一个中间件。

如果当前中间件没有终结请求-响应循环，则必须调用 `next()` 方法将控制权交给下一个中间件，否则请求就会挂起。

使用可选则挂载路径，可在应用级别或路由级别装载中间件。可装载一系列中间件函数，在挂载点创建一个中间件系统栈。

Express 应用可使用如下几种中间件：

- 应用级中间件
- 路由级中间件
- 错误处理中间件
- 内置中间件
- 第三方中间件

## 应用级中间件

应用级中间件绑定到 `express` 实例，使用 `app.use()` 和 `app.VERB()`。

```
var app = express();

// 没有挂载路径的中间件，应用的每个请求都会执行该中间件
app.use(function (req, res, next) {
  console.log('Time:', Date.now());
  next();
});

// 挂载至 /user/:id 的中间件，任何指向 /user/:id 的请求都会执行它
app.use('/user/:id', function (req, res, next) {
  console.log('Request Type:', req.method);
  next();
});

// 路由和句柄函数(中间件系统)，处理指向 /user/:id 的 GET 请求
app.get('/user/:id', function (req, res, next) {
  res.send('USER');
});
```

下面这个例子展示了在一个挂载点装载一组中间件。

```
// 一个中间件栈，对任何指向 /user/:id 的 HTTP 请求打印出相关信息
app.use('/user/:id', function(req, res, next) {
  console.log('Request URL:', req.originalUrl);
  next();
}, function (req, res, next) {
  console.log('Request Type:', req.method);
  next();
});
```

作为中间件系统的路由句柄，使得为路径定义多个路由成为可能。在下面的例子中，为指向 `/user/:id` 的 GET 请求定义了两个路由。第二个路由虽然不会带来任何问题，但却永远不会被调用，因为第一个路由已经终止了请求-响应循环。

```
// 一个中间件栈，处理指向 /user/:id 的 GET 请求
app.get('/user/:id', function (req, res, next) {
  console.log('ID:', req.params.id);
  next();
}, function (req, res, next) {
  res.send('User Info');
});

// 处理 /user/:id， 打印出用户 id
app.get('/user/:id', function (req, res, next) {
  res.end(req.params.id);
});
```

如果需要在中间件栈中跳过剩余中间件，调用 `next('route'` 方法将控制权交给下一个路由。需要注意的是 `next('route')` 只对使用 `app.VERB()` 或 `router.VERB()` 加载的中间件有效。

```
// 一个中间件栈，处理指向 /user/:id 的 GET 请求
app.get('/user/:id', function (req, res, next) {
  // 如果 user id 为 0, 跳到下一个路由
  if (req.params.id == 0) next('route');
  // 负责将控制权交给栈中下一个中间件
  else next(); // 
}, function (req, res, next) {
  // 渲染常规页面
  res.render('regular');
});

// 处理 /user/:id， 渲染一个特殊页面
app.get('/user/:id', function (req, res, next) {
  res.render('special');
});
```

## 路由级中间件

路由级中间件和应用级中间件一样，只是它绑定的对象为 `express.Router()`。

```
var router = express.Router();
```

路由级使用 `router.use()` 或 `router.VERB()` 加载。

上述在应用级创建的中间件系统，可通过如下代码改写为路由级：

```
var app = express();
var router = express.Router();

// 没有挂载路径的中间件，通过该路由的每个请求都会执行该中间件
router.use(function (req, res, next) {
  console.log('Time:', Date.now());
  next();
});

// 一个中间件栈，显示任何指向 /user/:id 的 HTTP 请求的信息
router.use('/user/:id', function(req, res, next) {
  console.log('Request URL:', req.originalUrl);
  next();
}, function (req, res, next) {
  console.log('Request Type:', req.method);
  next();
});

// 一个中间件栈，处理指向 /user/:id 的 GET 请求
router.get('/user/:id', function (req, res, next) {
  // 如果 user id 为 0, 跳到下一个路由
  if (req.params.id == 0) next('route');
  // 负责将控制权交给栈中下一个中间件
  else next(); // 
}, function (req, res, next) {
  // 渲染常规页面
  res.render('regular');
});

// 处理 /user/:id， 渲染一个特殊页面
router.get('/user/:id', function (req, res, next) {
  console.log(req.params.id);
  res.render('special');
});

// 将路由挂载至应用
app.use('/', router);
```

## 错误处理中间件

错误处理中间件有 4 个参数，定义错误处理中间件时必须使用这 4 个参数。即使不需要 `next` 对象，也必须在签名中声明它，否则中间件会被识别为一个常规中间件，不能处理错误。

错误处理中间件和其他中间件定义类似，只是要使用 4 个参数，而不是 3 个，其签名如下： `(err, req, res, next)`

```
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

请参考[错误处理](error-handling.md)一章了解更多关于错误处理中间件的内容。

## 内置中间件

在 4.x 版本中，Express 已经不再依赖 Connect。除了 `express.static`，Express 以前包括的中间件现在已经在一个单独的库里，请参考[中间件列表](https://github.com/senchalabs/connect?&&_ga=1.169674273.1286277039.1432108056#middleware)。


### express.static(root, [options])

`express.static` 是 Express 唯一内置的中间件，它基于 [serve-static](https://github.com/expressjs/serve-static?_ga=1.226969245.1286277039.1432108056)，负责在 Express 应用中提供静态资源。

参数 `root` 指提供静态资源的根目录。

可选的 `options` 参数拥有如下属性。

<table>
<thead>
<tr>
<th>属性</th>
<th>描述</th>
<th>类型</th>
<th>缺省值</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>dotfiles</code></td>
<td>是否提供 dotfiles，可选值为 “allow”、“deny” 和 “ignore”</td>
<td>String</td>
<td>“ignore”</td>
</tr>
<tr>
<td><code>etag</code></td>
<td>是否启用 etag 生成</td>
<td>Boolean</td>
<td><code>true</code></td>
</tr>
<tr>
<td><code>extensions</code></td>
<td>设置文件扩展回退</td>
<td>Boolean</td>
<td><code>false</code></td>
</tr>
<tr>
<td><code>index</code></td>
<td>发送目录索引文件，设置为 <code>false</code> 禁用目录索引。</td>
<td>Mixed</td>
<td>“index.html”</td>
</tr>
<tr>
<td><code>lastModified</code></td>
<td>设置 <code>Last-Modified</code> 头为文件在操作系统上的最后修改日期。可能值为 <code>true</code> 或 <code>false</code>。</td>
<td>Boolean</td>
<td><code>true</code></td>
</tr>
<tr>
<td><code>maxAge</code></td>
<td>以毫秒或者其<a href="https://www.npmjs.org/package/ms">字符串格式</a>设置 Cache-Control 头的 max-age 属性。</a></td>
<td>Number</td>
<td>0</td>
</tr>
<tr>
<td><code>redirect</code></td>
<td>当路径为目录时，重定向至 “/”。</td>
<td>Boolean</td>
<td><code>true</code></td>
</tr>
<tr>
<td><code>setHeaders</code></td>
<td>设置 HTTP 头以提供文件的函数。</td>
<td>Function</td>
<td>&nbsp;</td>
</tr>
</tbody>
</table>

下面的例子使用了 `express.static` 中间件，其中的 `options` 对象经过了精心的设计。

```
var options = {
  dotfiles: 'ignore',
  etag: false,
  extensions: ['htm', 'html'],
  index: false,
  maxAge: '1d',
  redirect: false,
  setHeaders: function (res, path, stat) {
    res.set('x-timestamp', Date.now());
  }
}

app.use(express.static('public', options));
```

每个应用可有多个静态目录。

```
app.use(express.static('public'));
app.use(express.static('uploads'));
app.use(express.static('files'));
```

请参考 [serve-static](https://github.com/expressjs/serve-static?_ga=1.226969245.1286277039.1432108056) 文档，了解更多关于 `serve-static` 和其选项的信息。

## 第三方中间件

Express 是一款提供路由和中间件的 Web 框架，但其本身的功能却异常精简。Express 应用的功能通过第三方中间件来添加。

安装所需功能的 node 模块，并在应用中加载，可以在应用级加载，也可以在路由级加载。

下面的例子安装并加载了一个解析 cookie 的中间件： `cookie-parser`

```
$ npm install cookie-parser
```

```
var express = require('express');
var app = express();
var cookieParser = require('cookie-parser');

// 加载 cookie 解析中间件
app.use(cookieParser());
```

请参考 [Third-party middleware](http://expressjs.com/resources/middleware.html) 获取 Express 中经常用到的第三方中间件列表。