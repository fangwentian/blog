---
title: nodejs常用小工具介绍
date: 2017/6/11
abbrlink: 62753
category:
- nodejs
tags:
- config
- chalk
- fs-extra
- opn
---

本文介绍了在node开发中非常高频率使用的几个小工具。

 - config
 - chalk
 - opn
 - fs-extra
 - path

#### config
&emsp;&emsp;在使用 Node.js 编写一个完整的项目时，程序中往往需要用到一些可配置的变量。

&emsp;&emsp;config模块是 NPM 上下载量最高的 Node.js 配置文件管理模块。它通过环境变量NODE_CONFIG_DIR来指定配置文件所在的目录，默认为./config（即当前运行目录下的config目录），通过环境变量NODE_ENV来指定当前的运行环境版本。

配置文件的后缀可以是.yml, .yaml, .xml, .coffee, .cson, .properties, .json, .json5, .hjson 或 .js, 通常用json  

&emsp;&emsp;模块加载后，会首先载入默认的配置文件${NODE_CONFIG_DIR}/default.json，再载入文件${NODE_CONFIG_DIR}/${NODE_ENV}.json，如果配置项有冲突则覆盖默认的配置（arrays are merged by replacement）。

&emsp;&emsp;默认的NODE_ENV是development

&emsp;&emsp;例如在config文件夹下，有三个文件，分别是config.json, development.json, production.json

```
- default.json: 
{
    "Customer": {
        "dbConfig": {
            "host": "localhost",
            "port": 5984,
            "dbName": "customers"
        },
        "credit": {
            "initialLimit": 100,
            "initialDays": 1
        }
    }
}

- development.json
{
    "Customer": {
        "dbConfig": {
            "host": "prod-db-server"
        },
        "credit": {
            "initialDays": 30
        }
    }
}

- production.json
{
    "Customer": {
        "credit": {
            "initialDays": 60
        }
    }
}

```

再新建app.js：
```
const CONFIG = require('config');

console.log(CONFIG.get('Customer.credit.initialDays'))
```

**输出结果为：30**

环境中如果没有设置NODE_ENV, 那么默认的会使用development， 因此development.json中的数据会覆盖default.json

如果运行：export NODE_ENV=production && node app.js  

**则输出结果为：60**

载入config模块后，其返回的对象实际上就是当前的配置信息，同时提供了两个方法get()和has()来操作配置项。比如：

```
const config = require('config');
console.log(config);
console.log(config.get('Customer'));
console.log(config.get('Customer.dbConfig'));
console.log(config.has('Customer.dbConfig.host'));
console.log(config.has('Customer.dbConfig.host2'));
```
其中get()用来获取指定配置，可以使用诸如Customer.dbConfig这样的格式，如果配置项不存在则会抛出异常。has()用来检测指定配置项是否存在，如果存在则返回true。

#### chalk
chalk对应中文粉笔，它希望做到的是彩色粉笔。

<center>
![](http://haitao.nos.netease.com/205a32ca1c034d73a8df95c53c0bf0e5.png)
</center>

chalk是一个让开发者能在控制台输出彩色信息的小工具，如果你之前用过color.js，那么可以认为chalk是color.js的替代者。

chalk的语法很简单，如下官方的示例：

```
const chalk = require('chalk');
const log = console.log;

// combine styled and normal strings
log(chalk.blue('Hello') + 'World' + chalk.red('!'));

// compose multiple styles using the chainable API
log(chalk.blue.bgRed.bold('Hello world!'));

// pass in multiple arguments
log(chalk.blue('Hello', 'World!', 'Foo', 'bar', 'biz', 'baz'));

// nest styles
log(chalk.red('Hello', chalk.underline.bgBlue('world') + '!'));

// nest styles of the same type even (color, underline, background)
log(chalk.green(
    'I am a green line ' +
    chalk.blue.underline.bold('with a blue substring') +
    ' that becomes green again!'
));

// ES2015 template literal
log(`
CPU: ${chalk.red('90%')}
RAM: ${chalk.green('40%')}
DISK: ${chalk.yellow('70%')}
`);
```
输出结果如下：

![](http://haitao.nos.netease.com/be37b6f7cfda45649f111f603c1b654d.png)

#### opn
opn是一个跨平台的，用node打开**文件，url或者可执行文件**的工具。

在开发模式下，node server启来之后，直接打开浏览器的127.0.0.1，是不是很方便的一个功能呢。

opn的用法：

```
const opn = require('opn');

// Opens the image in the default image viewer
opn('chalk.png').then(() => {
    console.log('open image success');
});

// Opens the url in the default browser
opn('http://kaola.com');

// Specify the app to open in
opn('http://kaola.com', {app: 'firefox'});
```
这样就分别打开了当前路径下的chalk.png文件，用默认的浏览器打开一个url，和指定浏览器打开url


#### fs-extra
fs-extra是系统fs模块的扩展，提供了更多便利的API。所有原生fs里的方法都没有改动的移植到了fs-extra。

所以，我们不需要再引入fs模块：
```
const fs = require('fs') // this is no longer necessary
```
这样就可以了：
```
const fs = require('fs-extra')
```
>fs-extra提供的API都分为sync（同步）和async（异步）。大多数的方法默认是异步的。

例如：
```
const fs = require('fs-extra')

fs.copy('/tmp/myfile', '/tmp/mynewfile', err => {
  if (err) return console.error(err)
  console.log("success!")
});

try {
  fs.copySync('/tmp/myfile', '/tmp/mynewfile')
  console.log("success!")
} catch (err) {
  console.error(err)
}
```

**部分fs-extra API介绍（以下API都为async，其sync API为async API名字后加Sync）：**

**copy**  

复制文件
copy(src, dest, [options], callback)

Example：
```
const fs = require('fs-extra')

fs.copy('/tmp/myfile', '/tmp/mynewfile', err => {
  if (err) return console.error(err)

  console.log('success!')
}) // copies file

fs.copy('/tmp/mydir', '/tmp/mynewdir', err => {
  if (err) return console.error(err)

  console.log('success!')
}) // cop
```

**emptyDir**

emptyDir(dir, [callback])

保证文件夹是空的，如果文件夹不为空，删除里面的文件。如果文件夹不存在，创建一个空文件夹。

Example：
```
const fs = require('fs-extra')

// assume this directory has a lot of files and folders
fs.emptyDir('/tmp/some/dir', err => {
  if (err) return console.error(err)

  console.log('success!')
})
```

**ensureDir**

ensureDir(dir, callback)

保证文件夹存在，如果文件夹不存在就创建一个。

Example：
```
const fs = require('fs-extra')

const dir = '/tmp/this/path/does/not/exist'
fs.ensureDir(dir, err => {
  console.log(err) // => null
  // dir has now been created, including the directory it is to be placed in
})
```

**ensureLink**

ensureLink(srcpath, dstpath, callback)

保证文件路径存在，如果没有这个文件结构，就创建。

Example：
```
const fs = require('fs-extra')

const srcpath = '/tmp/file.txt'
const dstpath = '/tmp/this/path/does/not/exist/file.txt'
fs.ensureLink(srcpath, dstpath, err => {
  console.log(err) // => null
  // link has now been created, including the directory it is to be placed in
})
```

**readJson**

readJson(file, [options], callback)

读一个json文件，并转换输出为一个对象。

Example：
```
const fs = require('fs-extra')

fs.readJson('./package.json', (err, packageObj) => {
  if (err) console.error(err)
  
  console.log(packageObj.version) // => 0.1.3
})
```

**writeJson**

writeJson(file, object, [options], callback)

将一个对象，写入到json文件中。

Example：
```
const fs = require('fs-extra')

fs.writeJson('./package.json', {name: 'fs-extra'}, err => {
  if (err) return console.error(err)

  console.log('success!')
})
```

完整的API[看这里](https://github.com/jprichardson/node-fs-extra)


#### path
nodejs原生path模块，使用频率非常高的一个模块，这里还是再介绍一下。

下面一一介绍其API：

**path.basename(path[, ext])**

返回path的最后一部分, 返回的结果可排除ext后缀字符串。

Example
```
path.basename('/foo/bar/baz/asdf/quux.html')
// Returns: 'quux.html'

path.basename('/foo/bar/baz/asdf/quux.html', '.html')
// Returns: 'quux'
```
**path.delimiter**  

提供平台具体的路径分隔符。

 - ; for Windows
 - : for POSIX

**path.dirname(path)**

返回path所在的文件夹的名称。

Example
```
path.dirname('/foo/bar/baz/asdf/quux')
// Returns: '/foo/bar/baz/asdf'
path.dirname('/foo/bar/baz/asdf/quux/')
// Returns: '/foo/bar/baz/asdf'
path.dirname('/foo/bar/baz/asdf/quux.html')
// Returns: '/foo/bar/baz/asdf'
```

**path.extname(path)**

方法返回path的后缀名。在path的最后一部分，从最后一个.到path的结尾。如果path的最后一部分没有.或者basename以.开始，返回空string

Example
```
path.extname('index.html')
// Returns: '.html'

path.extname('index.coffee.md')
// Returns: '.md'

path.extname('index.')
// Returns: '.'

path.extname('index')
// Returns: ''

path.extname('.index')
// Returns: ''
```

**path.isAbsolute(path)**

确定path是否是一个绝对路径

For example on POSIX:
```
path.isAbsolute('/foo/bar') // true
path.isAbsolute('/baz/..')  // true
path.isAbsolute('qux/')     // false
path.isAbsolute('.')        // false
```
On Windows:
```
path.isAbsolute('//server')    // true
path.isAbsolute('\\\\server')  // true
path.isAbsolute('C:/foo/..')   // true
path.isAbsolute('C:\\foo\\..') // true
path.isAbsolute('bar\\baz')    // false
path.isAbsolute('bar/baz')     // false
path.isAbsolute('.')           // false
```

**path.join([...paths])**

把各个路径端，用平台的分隔符连接起来，然后再normalizes，关于normalizes请看后面的API。

For example:
```
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..')
// Returns: '/foo/bar/baz/asdf'

path.join('foo', {}, 'bar')
// throws TypeError: Arguments to path.join must be strings
```

**path.normalize(path)**

将不符合规范的路径格式化，简化开发人员中处理各种复杂的路径判断，path中的.或者..会resolve

如果有多个连续路径分隔符，会被替换为一个路径分隔符。

For example on POSIX:
```
path.normalize('/foo/bar//baz/asdf/quux/..')
// Returns: '/foo/bar/baz/asdf'
```

On Windows:
```
path.normalize('C:\\temp\\\\foo\\bar\\..\\');
// Returns: 'C:\\temp\\foo\\'
```

**path.resolve([...paths])**

可以更简单的理解为不断的调用cd命令

For example on POSIX:
```
path.resolve('/foo/bar', './baz')
// cd '/foo/bar'
// cd './baz'
// Returns: '/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/')
// cd '/foo/bar'
// cd '/tmp/file/'
// Returns: '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
// if the current working directory is /home/myself/node,
// this returns '/home/myself/node/wwwroot/static_files/gif/image.gif'
```

**path.relative(from, to)**

返回从from到to的相对路径

For example on POSIX:
```
path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb')
// Returns: '../../impl/bbb'
```

On Windows:
```
path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb')
// Returns: '..\\..\\impl\\bbb'
```