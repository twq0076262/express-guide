# 迁移到 Express 4

## 概览

Express 4 是对 Express 3 的一个颠覆性改变，也就是说如果您更新了 Express， Express 3 应用会无法工作。

该章包含如下内容：

- Express 4 的变化。
- 一个从 Express 3 迁移到 Express 4 的示例。
- 迁移到 Express 4 应用生成器。

## Express 4 的变化

Express 4 的主要变化如下：

- 对 Express 内核和中间件系统的改进。不再依赖 Connect 和内置的中间件，您需要自己添加中间件。
- 对路由系统的改进。
- 其他变化。

其他变化请参考：

- [4.x 中的新功能](https://github.com/strongloop/express/wiki/New-features-in-4.x?_ga=1.223169179.1286277039.1432108056)
- [从 3.x 迁移到 4.x](https://github.com/strongloop/express/wiki/Migrating-from-3.x-to-4.x?_ga=1.223169179.1286277039.1432108056)

### 对 Express 内核和中间件系统的改进

Express 4 不再依赖 Connect，而且从内核中移除了除 `express.static` 外的所有内置中间件。也就是说现在的 Express 是一个独立的路由和中间件 Web 框架，Express 的版本升级不再受中间件更新的影响。

移除了内置的中间件后，您必须显式地添加所有运行应用需要的中间件。请遵循如下步骤：

1. 安装模块：`npm install --save <module-name>`
2. 在应用中引入模块：`require('module-name')`
3. 按照文档的描述使用模块：`app.use( ... )`

下表列出了 Express 3 和 Express 4 中对应的中间件。

<table class="doctable" border="1">
<tbody><tr><th>Express 3</th><th>Express 4</th></tr>
<tr><td><code>express.bodyParser</code></td>
<td><a href="https://github.com/expressjs/body-parser">body-parser</a> +
<a href="https://github.com/expressjs/multer">multer</a></td></tr>
<tr><td><code>express.compress</code></td>
<td><a href="https://github.com/expressjs/compression">compression</a></td></tr>
<tr><td><code>express.cookieSession</code></td>
<td><a href="https://github.com/expressjs/cookie-session">cookie-session</a></td></tr>
<tr><td><code>express.cookieParser</code></td>
<td><a href="https://github.com/expressjs/cookie-parser">cookie-parser</a></td></tr>
<tr><td><code>express.logger</code></td>
<td><a href="https://github.com/expressjs/morgan">morgan</a></td></tr>
<tr><td><code>express.session</code></td>
<td><a href="https://github.com/expressjs/session">express-session</a></td></tr>
<tr><td><code>express.favicon</code></td>
<td><a href="https://github.com/expressjs/serve-favicon">serve-favicon</a></td></tr>
<tr><td><code>express.responseTime</code></td>
<td><a href="https://github.com/expressjs/response-time">response-time</a></td></tr>
<tr><td><code>express.errorHandler</code></td>
<td><a href="https://github.com/expressjs/errorhandler">errorhandler</a></td></tr>
<tr><td><code>express.methodOverride</code></td>
<td><a href="https://github.com/expressjs/method-override">method-override</a></td></tr>
<tr><td><code>express.timeout</code></td>
<td><a href="https://github.com/expressjs/timeout">connect-timeout</a></td></tr>
<tr><td><code>express.vhost</code></td>
<td><a href="https://github.com/expressjs/vhost">vhost</a></td></tr>
<tr><td><code>express.csrf</code></td>
<td><a href="https://github.com/expressjs/csurf">csurf</a></td></tr>
<tr><td><code>express.directory</code></td>
<td><a href="https://github.com/expressjs/serve-index">serve-index</a></td></tr>
<tr><td><code>express.static</code></td>
<td><a href="https://github.com/expressjs/serve-static">serve-static</a></td></tr>
</tbody></table>

这里是 Express 4 的[所有中间件列表](https://github.com/senchalabs/connect#middleware)。

多数情况下，您可以直接使用 Express 4 中对应的中间件替换 Express 3 中的中间件，请参考 GitHub 中的模块文档了解更多信息。

**app.use accepts parameters**

在 Express 4 中，可以从路由句柄中读取参数，以该参数的值作为路径加载中间件，比如像下面这样：

```
app.use('/book/:id', function(req, res, next) {
  console.log('ID:', req.params.id);
  next();
});
```

### 路由系统

应用现在隐式地加载路由中间件，因此不需要担心中间件加载顺序。

定义路由的方式依然未变，但是新的路由系统有两个新功能能帮助您组织路由：

- 新方法 `app.route()` 可以为路由路径创建链式路由句柄。
- 新类 `express.Router` 可以创建可挂载的模块化路由句柄。

**app.route() 方法**

新增加的 `app.route()` 方法可为路由路径创建链式路由句柄。由于路径在一个地方指定，会让路由更加模块化，也能减少代码冗余和拼写错误。请参考 [Router() 文档](http://expressjs.com/4x/api.html#router)获取更多关于路由的信息。

下面是一个使用 `app.route()` 方法定义链式路由句柄的例子。

```
app.route('/book')
  .get(function(req, res) {
    res.send('Get a random book');
  })
  .post(function(req, res) {
    res.send('Add a book');
  })
  .put(function(req, res) {
    res.send('Update the book');
  });
```

**express.Router 类**

另外一个帮助组织路由的是新加的 `express.Router` 类，可使用它创建可挂载的模块化路由句柄。`Router` 类是一个完整的中间件和路由系统，鉴于此，人们常称之为“迷你应用”。

下面的例子创建了一个模块化的路由，并加载了一个中间件，然后定义了一些路由，并且在主应用中将其挂载到指定路径。

在应用目录下创建文件 `birds.js`，其内容如下：

```
var express = require('express');
var router = express.Router();

// 特针对于该路由的中间件
router.use(function timeLog(req, res, next) {
  console.log('Time: ', Date.now());
  next();
});
// 定义主页路由
router.get('/', function(req, res) {
  res.send('Birds home page');
});
// 定义 about 页面路由
router.get('/about', function(req, res) {
  res.send('About birds');
});

module.exports = router;
```

在应用中加载该路由：

```
var birds = require('./birds');
...
app.use('/birds', birds);
```

应用现在就可以处理发送到 `/birds` 和 `/birds/about` 的请求，并且会调用特针对于该路由的 `timeLog` 中间件。

## 其他变化

下表列出了 Express 4 中其他一些尽管不大，但是非常重要的变化。


<table class="doctable" border="1">
<tbody><tr>
<th>对象</th>
<th>描述</th>
</tr>
<tr>
<td>Node</td>
<td>Express 4 需要 Node 0.10.x 或以上版本，已经放弃了对
Node 0.8.x 的支持。</td>
</tr>
<tr>
<td>
<p><code>http.createServer()</code></p>
</td>
<td>
<p><已经不再需要 code>http</code> 模块，除非您需要直接使用它（socket.io/SPDY/HTTPS），使用
<code>app.listen()</code> 启动应用。</p>
</td>
</tr>
<tr>
<td>
<p><code>app.configure()</code></p>
</td>
<td>
<p>已经删除 <code>app.configure()</code>，使用
<code>process.env.NODE_ENV</code> 或者
<code>app.get('env')</code> 检测环境并配置应用。</p>
</td>
</tr>
<tr>
<td>
<p><code>json spaces</code></p>
</td>
<td>
<p>Express 4 默认禁用 <code>json spaces</code> 属性。</p>
</td>
</tr>
<tr>
<td>
<p><code>req.accepted()</code></p>
</td>
<td>
<p>使用 <code>req.accepts()</code>、 <code>req.acceptsEncodings()</code>、
<code>req.acceptsCharsets()</code> 和 <code>req.acceptsLanguages()</code>。</p>
</td>
</tr>
<tr>
<td>
<p><code>res.location()</code></p>
</td>
<td>
<p>不再解析相对 URLs。</p>
</td>
</tr>
<tr>
<td>
<p><code>req.params</code></p>
</td>
<td>
<p>从数组变为对象。</p>
</td>
</tr>
<tr>
<td>
<p><code>res.locals</code></p>
</td>
<td>
<p>从函数变为对象。</p>
</td>
</tr>
<tr>
<td>
<p><code>res.headerSent</code></p>
</td>
<td>
<p>变为 <code>res.headersSent</code>。</p>
</td>
</tr>
<tr>
<td>
<p><code>app.route</code></p>
</td>
<td>
<p>变为 <code>app.mountpath</code>。</p>
</td>
</tr>
<tr>
<td>
<p><code>res.on('header')</code></p>
</td>
<td>
<p>已删除。</p>
</td>
</tr>
<tr>
<td>
<p><code>res.charset</code></p>
</td>
<td>
<p>已删除。</p>
</td>
</tr>
<tr>
<td>
<p><code>res.setHeader('Set-Cookie', val)</code></p>
</td>
<td>
<p>功能仅限于设置基本的 cookie 值，使用
<code>res.cookie()</code> 访问更多功能。</p>
</td>
</tr>
</tbody></table>


## 迁移示例

下面是一个从 Express 3 迁移到 Express 4 的例子，请留意 `app.js` 和 `package.json`。

**Express 3 应用**

app.js

请看如下 Express 3 应用，其 `app.js` 文件内容如下：

```
var express = require('express');
var routes = require('./routes');
var user = require('./routes/user');
var http = require('http');
var path = require('path');

var app = express();

// 环境
app.set('port', process.env.PORT || 3000);
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');
app.use(express.favicon());
app.use(express.logger('dev'));
app.use(express.methodOverride());
app.use(express.session({ secret: 'your secret here' }));
app.use(express.bodyParser());
app.use(app.router);
app.use(express.static(path.join(__dirname, 'public')));

// 只为开发使用
if ('development' == app.get('env')) {
  app.use(express.errorHandler());
}

app.get('/', routes.index);
app.get('/users', user.list);

http.createServer(app).listen(app.get('port'), function(){
  console.log('Express server listening on port ' + app.get('port'));
});
```

package.json

相应的 `package.json` 文件内容如下：

```
{
  "name": "application-name",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "3.12.0",
    "jade": "*"
  }
}
```

### 迁移过程

首先安装 Express 4 应用需要的中间件，使用如下命令将 Express 和 Jade 更新至最新版本：

```
$ npm install serve-favicon morgan method-override express-session body-parser multer errorhandler express@latest jade@latest --save
```

按如下方式修改 `app.js` 文件：

1. `express.favicon`、`express.logger`、`express.methodOverride`、`express.session`、`express.bodyParser`、`express.errorHandler` 这些内置中间件在 `express` 对象中已经没有了，您必须手动安装相应的中间件，并在应用中加载它们。
2. 不需要加载 `app.router`，它不再是一个合法的 Express 4 对象，删掉 `app.use(app.router);`。
3. 确保加载中间件的顺序正确，加载完应用路由后再加载 `errorHandler`。

**Express 4 应用**

package.json

运行上述 npm 命令后，会将 `package.json` 文件更新为：

```
{
  "name": "application-name",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "body-parser": "^1.5.2",
    "errorhandler": "^1.1.1",
    "express": "^4.8.0",
    "express-session": "^1.7.2",
    "jade": "^1.5.0",
    "method-override": "^2.1.2",
    "morgan": "^1.2.2",
    "multer": "^0.1.3",
    "serve-favicon": "^2.0.1"
  }
}
```

app.js

删掉非法的代码，加载需要的中间件，再做一些必要的修改，新的 `app.js` 内容如下：

```
var http = require('http');
var express = require('express');
var routes = require('./routes');
var user = require('./routes/user');
var path = require('path');

var favicon = require('serve-favicon');
var logger = require('morgan');
var methodOverride = require('method-override');
var session = require('express-session');
var bodyParser = require('body-parser');
var multer = require('multer');
var errorHandler = require('errorhandler');

var app = express();

// 环境
app.set('port', process.env.PORT || 3000);
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');
app.use(favicon(__dirname + '/public/favicon.ico'));
app.use(logger('dev'));
app.use(methodOverride());
app.use(session({ resave: true,
                  saveUninitialized: true,
                  secret: 'uwotm8' }));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(multer());
app.use(express.static(path.join(__dirname, 'public')));

app.get('/', routes.index);
app.get('/users', user.list);

// 加载路由完成后才能加载错误处理中间件
if ('development' == app.get('env')) {
  app.use(errorHandler());
}

var server = http.createServer(app);
server.listen(app.get('port'), function(){
  console.log('Express server listening on port ' + app.get('port'));
});
```

除非需要直接使用 `http` 模块（socket.io/SPDY/HTTPS），否则不必加载它，可使用如下方式启动应用：

```
app.listen(app.get('port'), function(){
  console.log('Express server listening on port ' + app.get('port'));
});
```

### 运行应用

迁移完成后，应用就变成了 Express 4 应用。为了确保迁移成功，使用如下命令启动应用：

```
$ node .
```

输入 [http://localhost:3000](http://localhost:3000)，即可看到经由 Express 4 渲染的主页。


## 迁移到 Express 4 应用生成器

生成 Express 应用的命令行还是 `express`，为了升级到最新版本，您必须首先卸载 Express 3 的应用生成器，然后安装新的 `express-generator`。

### 安装

如果您已经安装了 Express 3 应用生成器，请使用如下命令卸载：

```
$ npm uninstall -g express
```

根据您的文件目录权限，您可能需要以 `sudo` 权限执行该命令。

然后安装新的生成器：

```
$ npm install -g express-generator
```

根据您的文件目录权限，您可能需要以 `sudo` 权限执行该命令。

现在系统的 `express` 命令就升级为 Express 4 应用生成器了。

### 应用生成器的变化

大部分命令参数和使用方法都维持不变，除过如下选项：

- 删掉了 `--sessions` 选项。
- 删掉了 `--jshtml` 选项。
- 增加了 `--hogan` 选项以支持 [Hogan.js](http://twitter.github.io/hogan.js/?_ga=1.132124046.1286277039.1432108056)。

### 示例

运行下述命令创建一个 Express 4 应用：

```
$ express app4
```

如果查看 `app4/app.js` 的内容，会发现应用需要的所有中间件（不包括 `express.static`）都作为独立模块载入，而且再不显式地加载 `router` 中间件。

您可能还会发现，和旧的生成器生成的应用相比， `app.js` 现在成了一个 Node 模块。

安装完依赖后，使用如下命令启动应用：

```
$ npm start
```

如果看一看 `package.json` 文件中的 npm 启动脚本，会发现启动应用的真正命令是 `node ./bin/www`，在 Express 3 中则为 `node app.js`。

Express 4 应用生成器生成的 `app.js` 是一个 Node 模块，不能作为应用（除非修改代码）单独启动，需要通过一个 Node 文件加载并启动，这里这个文件就是 `node ./bin/www`。


创建或启动 Express 应用时，`bin` 目录或者文件名没有后缀的 `www` 文件都不是必需的，它们只是生成器推荐的做法，请根据需要修改。

如果不想保留 `www`，想让应用变成 Express 3 的形式，则需要删除 `module.exports = app;`，并在 `app.js` 末尾粘贴如下代码。

```
app.set('port', process.env.PORT || 3000);

var server = app.listen(app.get('port'), function() {
  debug('Express server listening on port ' + server.address().port);
});
```

记得在 `app.js` 上方加入如下代码加载 `debug` 模块。

```
var debug = require('debug')('app4');
```

然后将 `package.json` 文件中的 `"start": "node ./bin/www"` 修改为 `"start": "node app.js"`。

现在就将 `./bin/www` 的功能又改回到 `app.js` 中了。我们并不推荐这样做，这个练习只是为了帮助大家理解 `./bin/www` 是如何工作的，以及为什么 `app.js` 不能再自己启动。
