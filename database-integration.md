
# 数据库集成

为 Express 应用添加连接数据库的能力，只需要加载相应数据库的 Node.js 驱动即可。这里将会简要介绍如何为 Express 应用添加和使用一些常用的数据库 Node 模块。

- Cassandra
- CouchDB
- LevelDB
- MySQL
- MongoDB
- Neo4j
- PostgreSQL
- Redis
- SQLite
- ElasticSearch

这些数据库驱动只是其中一部分，可在 [npm 官网](https://www.npmjs.com/) 查找更多驱动。

## Cassandra

**模块**: [cassandra-driver](https://github.com/datastax/nodejs-driver)  
**安装**

```
$ npm install cassandra-driver
```

**示例**

```
var cassandra = require('cassandra-driver');
var client = new cassandra.Client({ contactPoints: ['localhost']});

client.execute('select key from system.local', function(err, result) {
  if (err) throw err;
  console.log(result.rows[0]);
});
```

## CouchDB

**模块**: [nano](https://github.com/dscape/nano)  
**安装**

```
$ npm install nano
```

**示例**

```
var nano = require('nano')('http://localhost:5984');
nano.db.create('books');
var books = nano.db.use('books');

//在数据库 books 中插入一条图书记录
books.insert({name: 'The Art of war'}, null, function(err, body) {
  if (!err){
    console.log(body);
  }
});

//返回图书列表
books.list(function(err, body){
  console.log(body.rows);
}
```


## LevelDB

**模块**: [levelup](https://github.com/rvagg/node-levelup)  
**安装**

```
$ npm install level levelup leveldown
```

**示例**

```
var levelup = require('levelup');
var db = levelup('./mydb');

db.put('name', 'LevelUP', function (err) {

  if (err) return console.log('Ooops!', err);
  db.get('name', function (err, value) {
    if (err) return console.log('Ooops!', err);
    console.log('name=' + value)
  });

});
```


## MySQL

**模块**: [mysql](https://github.com/felixge/node-mysql/)  
**安装**

```
$ npm install mysql
```

**示例**

```
var mysql      = require('mysql');
var connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'dbuser',
  password : 's3kreee7'
});

connection.connect();

connection.query('SELECT 1 + 1 AS solution', function(err, rows, fields) {
  if (err) throw err;
  console.log('The solution is: ', rows[0].solution);
});

connection.end();
```


## MongoDB

**模块**: [mongoskin](https://github.com/kissjs/node-mongoskin)  
**安装**

```
$ npm install mongoskin
```

**示例**

```
var db = require('mongoskin').db('localhost:27017/animals');

db.collection('mamals').find().toArray(function(err, result) {
  if (err) throw err;
  console.log(result);
});
```

如果想获取 MongoDB 的对象模型驱动，请参考 [Mongoose](https://github.com/LearnBoost/mongoose)。


## Neo4j

**模块**: [apoc](https://github.com/hacksparrow/apoc)  
**安装**

```
$ npm install apoc
```

**示例**

```
var apoc = require('apoc');

apoc.query('match (n) return n').exec().then(
  function (response) {
    console.log(response);
  },
  function (fail) {
    console.log(fail);
  }
);
```

## PostgreSQL

**模块**: [pg](https://github.com/brianc/node-postgres)  
**安装**

```
$ npm install pg
```

**示例**

```
var pg = require('pg');
var conString = "postgres://username:password@localhost/database";

pg.connect(conString, function(err, client, done) {

  if (err) {
    return console.error('error fetching client from pool', err);
  }
  client.query('SELECT $1::int AS number', ['1'], function(err, result) {
    done();
    if (err) {
      return console.error('error running query', err);
    }
    console.log(result.rows[0].number);
  });

});
```

## Redis

**模块**: [redis](https://github.com/mranney/node_redis)  
**安装**

```
$ npm install redis
```

**示例**

```
var client = require('redis').createClient();

client.on('error', function (err) {
  console.log('Error ' + err);
});

client.set('string key', 'string val', redis.print);
client.hset('hash key', 'hashtest 1', 'some value', redis.print);
client.hset(['hash key', 'hashtest 2', 'some other value'], redis.print);

client.hkeys('hash key', function (err, replies) {

  console.log(replies.length + ' replies:');
  replies.forEach(function (reply, i) {
    console.log('    ' + i + ': ' + reply);
  });

  client.quit();

});
```


## SQLite

**模块**: [sqlite3](https://github.com/mapbox/node-sqlite3)  
**安装**

```
$ npm install sqlite3
```

**示例**

```
var sqlite3 = require('sqlite3').verbose();
var db = new sqlite3.Database(':memory:');

db.serialize(function() {

  db.run('CREATE TABLE lorem (info TEXT)');
  var stmt = db.prepare('INSERT INTO lorem VALUES (?)');

  for (var i = 0; i < 10; i++) {
    stmt.run('Ipsum ' + i);
  }

  stmt.finalize();

  db.each('SELECT rowid AS id, info FROM lorem', function(err, row) {
    console.log(row.id + ': ' + row.info);
  });
});

db.close();
```


## ElasticSearch

**模块**: [elasticsearch](https://github.com/elastic/elasticsearch-js)  
**安装**

```
$ npm install elasticsearch
```

**示例**

```
var elasticsearch = require('elasticsearch');
var client = elasticsearch.Client({
  host: 'localhost:9200'  
});

client.search({
  index: 'books',
  type: 'book',
  body: {
    query: {
      multi_match: {
        query: 'express js',
        fields: ['title', 'description']
      }
    }
  }
}).then(function(response) {
  var hits = reponse.hits.hits;
}, function(error) {
  console.trace(error.message);
});
```