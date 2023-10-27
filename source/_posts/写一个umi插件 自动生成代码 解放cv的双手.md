### 引言

最近在写一个中台项目，使用的`react`的`umi`框架。
各种增删改查。基本是列表页 新建页 详情页这种页面
为了避免不必要的简单重复（主要是想偷懒） 于是想去实现自己的一个代码生成器

### 简单探索

首先，在官网上看到了官方写的一个生成器

<img src="[https://images.cnblogs.com/cnblogs_com/amigod/1602334/o_探索umi-官网.png](https://images.cnblogs.com/cnblogs_com/amigod/1602334/o_%E6%8E%A2%E7%B4%A2umi-%E5%AE%98%E7%BD%91.png)"/ alt="官网图片">

再去源码里扒一扒 找到关键所在

![](https://cdn.nlark.com/yuque/0/2020/png/1512483/1591943686220-fd368c99-431d-4618-8e24-1c3acb51d679.png#height=167&id=sDLZY&originHeight=167&originWidth=1025&originalType=binary&ratio=1&rotation=0&showTitle=false&size=0&status=done&style=none&title=&width=1025)

简而言之，就是利用插件的`api`注册了一个生成model的指令，生成器指向目录里的`model.js`

代码如下

```javascript
import { join } from 'path';
import assert from 'assert';

export default api => {
  const { paths, config } = api;
  const absTemplatePath = join(__dirname, '../template/generators');

  return class Generator extends api.Generator {
    writing() {
       ...
       // 判断目录名是models还是model
      const models = config.singular ? 'model' : 'models';
      const name = this.args[0].toString();
      ...
     // 将模板目录下里的model代码 拷贝到项目的model目录下 并命名为指令输入的文件名
      this.fs.copyTpl(
        join(absTemplatePath, 'model.js'),
        join(paths.absSrcPath, models, `${name}.js`),
        {
          name,
        },
      );
    }
  };
};
```

```javascript
../template/generators/model.js

export default {
  state: '<%= name %>',
  subscriptions: {
    setup({ dispatch, history }) {
    },
  },
  reducers: {
    update(state) {
      return `${state}_<%= name %>`;
    },
  },
  effects: {
    *fetch({ type, payload }, { put, call, select }) {
    },
  },
}
```

model是一个常规的`dva`的`model`
里面的`<%= name %>`是`ejs`语法,对应着`copyTpl`方法的第三个参数中的`name`
模板js里的这个占位会被参数`name`替换

因为我们项目中习惯将model写到模块文件夹下，而且model里的代码有些我们的自己的书写
所以需要自定义一个生成方法了。

### 继续深入

虽然实现 但是还是带着一些疑问

- 如何注册到umi中去
- generator是基于第三方的生成器还是umi自带
- fs 又是用的是什么插件 如何运作的

#### umi如何整合的

在umi-build-dev库下的 PluginAPI里有这样一段代码

```javascript
import BasicGenerator from './BasicGenerator';
export default class PluginAPI {
  constructor(id, service) {
 .....................
    this.Generator = BasicGenerator;
  }
  registerGenerator(name, opts) {
    const { generators } = this.service;
    assert(typeof name === 'string', `name should be supplied with a string, but got ${name}`);
    assert(opts && opts.Generator, `opts.Generator should be supplied`);
    assert(!(name in generators), `Generator ${name} exists, please select another one.`);
    generators[name] = opts;
  }
..............
```

就是我们注册用的方法，这边一方便将BasicGenerator在实例化的时候 挂到Generator属性上
另一方吧提供了registerGenerator方法 也就是我们之前调用的，进行注册

```javascript
BasicGenerator //js
import Generator from 'yeoman-generator';
const { existsSync } = require('fs');
const { join } = require('path');

class BasicGenerator extends Generator {
  constructor(args, opts) {
    super(args, opts);
    this.isTypeScript = existsSync(join(opts.env.cwd, 'tsconfig.json'));
  }
}

export default BasicGenerator;
```

```javascript
// Service.js
export default class Service {
  constructor({ cwd }) {
    //  用户传入的 cmd 不可信任 转化一下
    this.cwd = cwd || process.cwd();

    try {
    ....
    this.generators = {};
    ....
```

发现generator只是一个接收数据的对象

这里顺便一提，umi插件中经常用到的api其实就是在service中用proxy属性代理了一下pluginAPI生成的
在初始化插件件方法 `initPlugin`中

```javascript
this是service对象
  const api = new Proxy(new PluginAPI(id, this), {
        get: (target, prop) => {
          if (this.pluginMethods[prop]) {
            return this.pluginMethods[prop];
          }
          if (
            [
              // methods
              'changePluginOption',
              'applyPlugins',
              '_applyPluginsAsync',
              'writeTmpFile',
              'getRoutes',
              'getRouteComponents',
              // properties
              'cwd',
              'config',
              'webpackConfig',
              'pkg',
              'paths',
              'routes',
              // error handler
              'UmiError',
              'printUmiError',
              // dev methods
              'restart',
              'printError',
              'printWarn',
              'refreshBrowser',
              'rebuildTmpFiles',
              'rebuildHTML',
            ].includes(prop)
          ) {
            if (typeof this[prop] === 'function') {
              return this[prop].bind(this);
            } else {
              return this[prop];
            }
          } else {
            return target[prop];
          }
        },
      });
```

大概意思就是对PluginAPI实例化后的属性进行get的代理 优先使用pluginMethods里注册的方法 其次是如果是数组总的方法，优先在service里找 最后才到PluignAPI

#### 指令注册和方法实现

代码入口:umi-build-dev/src/plugin/commnds 下的generate文件夹下

```javascript
export default function(api) {
  const {
    service: { generators },
    log,
  } = api;

  function generate(args = {}) {
    try {
      const name = args._[0];
      assert(name, `run ${chalk.cyan.underline('umi help generate')} to checkout the usage`);
      assert(generators[name], `Generator ${chalk.cyan.underline(name)} not found`);
      const { Generator, resolved } = generators[name];
      const generator = new Generator(args._.slice(1), {
        ...args,
        env: {
          cwd: api.cwd,
        },
        resolved: resolved || __dirname,
      });
      return generator
        .run()
        .then(() => {
          log.success('');
        })
        .catch(e => {
          log.error(e);
        });
    } catch (e) {
      log.error(`Generate failed, ${e.message}`);
      console.log(e);
    }
  }

  function registerCommand(command, description) {
    const details = `
Examples:

  ${chalk.gray('# generate page users')}
  umi generate page users

  ${chalk.gray('# g is the alias for generate')}
  umi g page index

  ${chalk.gray('# generate page with less file')}
  umi g page index --less
  `.trim();
    api.registerCommand(
      command,
      {
        description,
        usage: `umi ${command} type name [options]`,
        details,
      },
      generate,
    );
  }

  registerCommand('g', 'generate code snippets quickly (alias for generate)');
  registerCommand('generate', 'generate code snippets quickly');
```

这段代码关键在`new Generator`,Generator是我们注册的,但是`run`是哪来的呢,记得注册的`Generator extends api.Generator`,让我们来探索下这个`api.Generator`吧 
#### Generator 
稍微翻了一下代码 发现了generator的真面目`yeoman-generator`
这玩意是一个基于事件通知的队列函数执行流程

```javascript
run(cb) {
    const promise = new Promise((resolve, reject) => {
      const self = this;
      this._running = true;
      this.emit('run');

      const methods = Object.getOwnPropertyNames(Object.getPrototypeOf(this));
      const validMethods = methods.filter(methodIsValid);
      assert(
        validMethods.length,
        'This Generator is empty. Add at least one method for it to run.'
      );

      this.env.runLoop.once('end', () => {
        this.emit('end');
        resolve();
      });

      // Ensure a prototype method is a candidate run by default
      function methodIsValid(name) {
        return name.charAt(0) !== '_' && name !== 'constructor';
      }

      function addMethod(method, methodName, queueName) {
        queueName = queueName || 'default';
        debug(`Queueing ${methodName} in ${queueName}`);
        self.env.runLoop.add(queueName, completed => {
          debug(`Running ${methodName}`);
          self.emit(`method:${methodName}`);

          runAsync(function() {
            self.async = () => this.async();
            return method.apply(self, self.args);
          })()
            .then(completed)
            .catch(err => {
              debug(`An error occured while running ${methodName}`, err);

              // Ensure we emit the error event outside the promise context so it won't be
              // swallowed when there's no listeners.
              setImmediate(() => {
                self.emit('error', err);
                reject(err);
              });
            });
        });
      }

      function addInQueue(name) {
        const item = Object.getPrototypeOf(self)[name];
        const queueName = self.env.runLoop.queueNames.indexOf(name) === -1 ? null : name;

        // Name points to a function; run it!
        if (typeof item === 'function') {
          return addMethod(item, name, queueName);
        }

        // Not a queue hash; stop
        if (!queueName) {
          return;
        }

        // Run each queue items
        _.each(item, (method, methodName) => {
          if (!_.isFunction(method) || !methodIsValid(methodName)) {
            return;
          }

          addMethod(method, methodName, queueName);
        });
      }

      validMethods.forEach(addInQueue);

      const writeFiles = () => {
        this.env.runLoop.add('conflicts', this._writeFiles.bind(this), {
          once: 'write memory fs to disk'
        });
      };

      this.env.sharedFs.on('change', writeFiles);
      writeFiles();

      // Add the default conflicts handling
      this.env.runLoop.add('conflicts', done => {
        this.conflicter.resolve(err => {
          if (err) {
            this.emit('error', err);
          }

          done();
        });
      });

      _.invokeMap(this._composedWith, 'run');
    });

    // Maintain backward compatibility with the callback function
    if (_.isFunction(cb)) {
      promise.then(cb, cb);
    }

    return promise;
  }
```

这里用了Promise来进行流程控制

#### 关于fs

```javascript
// yeoman-generator
const FileEditor = require('mem-fs-editor');
class Generator extends EventEmitter {
  constructor(args, options) {
    super();
    ..........
    this.fs = FileEditor.create(this.env.sharedFs);
  }
```

```javascript
// mem-fs-editor
'use strict';

function EditionInterface(store) {
  this.store = store;
}

EditionInterface.prototype.read = require('./actions/read.js');
EditionInterface.prototype.readJSON = require('./actions/read-json.js');
EditionInterface.prototype.exists = require('./actions/exists');
EditionInterface.prototype.write = require('./actions/write.js');
EditionInterface.prototype.writeJSON = require('./actions/write-json.js');
EditionInterface.prototype.extendJSON = require('./actions/extend-json.js');
EditionInterface.prototype.append = require('./actions/append.js');
EditionInterface.prototype.delete = require('./actions/delete.js');
EditionInterface.prototype.copy = require('./actions/copy.js').copy;
EditionInterface.prototype._copySingle = require('./actions/copy.js')._copySingle;
EditionInterface.prototype.copyTpl = require('./actions/copy-tpl.js');
EditionInterface.prototype.move = require('./actions/move.js');
EditionInterface.prototype.commit = require('./actions/commit.js');

exports.create = function (store) {
  return new EditionInterface(store);
};
```

我们用到的copyTpl方法

```javascript
'use strict';

var extend = require('deep-extend');
var ejs = require('ejs');
var isBinaryFileSync = require('isbinaryfile').isBinaryFileSync;

function render(contents, filename, context, tplSettings) {
  let result;

  const contentsBuffer = Buffer.from(contents, 'binary');
  if (isBinaryFileSync(contentsBuffer, contentsBuffer.length)) {
    result = contentsBuffer;
  } else {
    result = ejs.render(
      contents.toString(),
      context,
      // Setting filename by default allow including partials.
      extend({filename: filename}, tplSettings)
    );
  }

  return result;
}

module.exports = function (from, to, context, tplSettings, options) {
  context = context || {};
  tplSettings = tplSettings || {};

  this.copy(
    from,
    to,
    extend(options || {}, {
      process: function (contents, filename) {
        return render(contents, filename, context, tplSettings);
      }
    }),
    context,
    tplSettings
  );
};
```

### 上手

以下是我写的一个生成规则

```javascript
import { join } from 'path';
const fs=require('fs');
export default api => {
  const  {paths} = api;
  const configPath=join(paths.absSrcPath,'generatorConfig.js');
  const absTemplatePath = join(__dirname, '../template/generators');
  return class Generator extends api.Generator {
    writing() {
      const name = this.args[0].toString();
      // assert(!name.includes('/'), `model name should not contains /, bug got ${name}`);
      const type =this.args[1]&& this.args[1].toString();
     // type即为命令后跟的参数
      switch (type) {
        case 'list':
          if(!fs.existsSync(configPath)) {
            api.log.error('新建列表模板缺少generatorConfig.js')
            return
          }
          const genConfig=require(configPath);
          this.fs.copyTpl(join(absTemplatePath, 'list.js'),join(paths.absSrcPath, `pages/${name}/${type}`, `index.js`), {
            name,
            queryFormItems:genConfig[name]['queryFormItems'],
            columns:genConfig[name]['columns']
          });
      }
      this.fs.copyTpl(join(absTemplatePath, 'model.js'), join(paths.absSrcPath, `pages/${name}`, `model.js`), {
        name
      });
      this.fs.copyTpl(join(absTemplatePath, 'index.less'), join(paths.absSrcPath, `pages/${name}`, `index.less`), {
        name
      });
      this.fs.copyTpl(join(absTemplatePath, 'service.js'), join(paths.absSrcPath, `pages/${name}`, `service.js`), {
        name
      });
    }
  };
};
```

添加了如下功能

- 结合项目中的目录结构约定进行目录生成（比如我们约定用service来进行接口方法管理）
- 增加在命令后面加不同参数 生成不同的特征模块（比如列表 详情）
- 增加了配置项 可以在node环境下去读取配置 再生成到代码里去（比如 `antd`的列表的`columns`）

再仿照`umi-dva-plugin`的流程进行`命令注册`和`插件导出`

```
import { join } from 'path';
export default(api, opts = {})=> {
  api.registerGenerator('dva:newPage', {
    Generator: require('./model').default(api),
    resolved: join(__dirname, './model'),
  });
}
```


### 流程示意

![](https://cdn.nlark.com/yuque/0/2022/jpeg/1512483/1645607000777-a41c6da1-b772-43a8-a50e-08f5e6a8e639.jpeg)


### 遇到问题

在探索和上手遇到挺多问题，总结如下
1.阅读源码 加以甄别 ，因为`umi-dva-plugin`的代码贼多，模板功能只是其中的非核心功能，所以也是看了好几遍 发现这个功能其实和其他代码并不存在耦合 可以单独提出来
2.探索模板语法 一开始不知道是`ejs` 找了下`copyTpl`方法
![](https://cdn.nlark.com/yuque/0/2020/png/1512483/1591943686860-58e3860d-fff7-4f46-af0a-dbfc5760430c.png#height=556&id=TUUnH&originHeight=556&originWidth=1590&originalType=binary&ratio=1&rotation=0&showTitle=false&size=0&status=done&style=none&title=&width=1590)
然后就恍然大悟，怪不得看起来那么熟悉,顺便学了一下ejs模板`<%= %>`和`<%- %>`的区别
3.兼容性问题 遇到的一个贼奇怪的问题 node环境兼容的问题
一开始不知道 用babel转成es5了  一直报错`class constructor Generator cannot be invoked without 'new`
看上去就是个兼容问题 然后用web版的babel转换器 关闭`preset es2015` 调整`node版本`到`6.4`主要是把对象的`解构赋值要转换掉` 不然依赖的三方`Generator`可能不认

### 总结


现在看来其实写这个插件其实并不难，但是在当时很多知识都不了解的情况下去看，确实还是有些许棘手，了解用法和原理比较有挑战，毕竟不是自己写的代码，所以还是要加强代码方便的阅读。

### 项目链接

[戳我查看](https://www.npmjs.com/package/dva-enhance)
