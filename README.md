> 本文是16年4月9号 深圳 Node Party 的讲稿修改而成。[活动总结](http://cnodejs.org/topic/570907402a1f230a25be8aab)

## Node.js 在广发证券：koa2 和 微服务实战新一代API Server

### 前言
#### 自我介绍

前几位的分享都特别棒，最后我给大家带来的topic是讲述我们广发证券这样一个传统券商在新技术如nodejs，微服务，koa上的一些尝试和使用经验吧。

先做个简单的自我介绍：我13年毕业，之前在百度实习做前端开发的一些工作，临近毕业去了豌豆荚这家创业公司做WebApp开发（主要集中在Angular使用上）。接下来从去年5月到目前所在的广发证券，先是做混合应用ionic相关开发，而现在主要是focus在Node.js在团队内部的使用和推广上。

私下我对技术还是非常感兴趣，结合自己思考和理解，会翻译和写一些文章在自己的[github的博客](http://github.com/gaohailang/blog)上。

 ![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598836453050.jpg)￼

#### 分享大纲

这是今天我要分享的大纲大致是：

- 先讲下我们技术实践产生的背景 - 即我们这个团队是谁
- 然后是我们今天的主角node.js的在团队内的选型定位
- 我们怎么看待要实施的API Server，看它的发展历史和未来展望
- 接着我会先后从 koa2相关的实践，普通的node开发进行讲诉，最后以微服务结尾。

// 如果时间允许会看下我们在开源的贡献和参与。


### 我们是谁


那我们是谁呢，广发证券，国内的TOP3的券商的信息技术部门。
从13年开始，我们已经重视相关技术的积累，我们标榜自己是一只fintech范的团队。
我们希望和国际投行对肩，『目前证券行业创新高涨尤其现在的互联网金融。
在国际化进程中，IT人员会占到 1/3的比例，国内远远不到，我们在这人员配置方面还在努力。
我们的技术选型时非常前言的，可以看到我们在13年就开始使用类似于angular, node.js等，算是非常早的运用这些技术框架开发复杂运用的公司了（金融领域

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598836936018.jpg)

为什么这么做了，激进的采用这样的方式，首先这些新的基于互联网的新业务上允许了而且需要迭代快开发效率高的技术。然后最主要还是人员上的思考。

我们需要这样的技术态度：

- 来吸引想在座爱玩技术的你们。
- 你们用这些新技术带我们弯道超车（要知道这些开源技术很多时候比原本自研的强多了，社区强大以后找工作也好找，偷笑）
- 最终我们致力于建立学习型组织在现在高速发展的技术世界保持前沿，而不是完全被陷入在复杂业务中

那么就不难理解我们目前技术栈选项的背后，那么究竟 Node.js 在我们公司技术栈到底是什么定位呢？

```note
// 去看我们的一些技术选项前，先看看我们是什么样的组织，再看看我们对开源技术的态度，这样才能得出一些背后的原因，也给你们一些技术参考。
//（我是较高复杂度的譬如购买一个理财产品很多逻辑的判断和，流量上的倒不是很大，但业务上流动的钱到时候百千万到亿的~，其次我们推崇的微服务就允许让我们xxx（因为它xxx
```


### 我们对 Node.js 的定位

#### 我们的技术全景图

这是我们的技术体系全景图，对它感兴趣的会后可以详细在看）。
我们先要看第二列云端/edge部分，就这里就是我们nodejs发光发热的部分（它在接入层非常灵活的对接前面各种终端入口请求，做他合适做的事情）
然后在后面与第三列的微服务结合，打通和连接背后的更后端的东西（譬如大数据，金融柜台，交易总线等等）。

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598838237638.jpg)￼￼


并且从去年开始，我们推崇从接入层之后到柜台之前的服务和业务都变微服务的形式来提供，在保证接口服务的健壮性的同时，提供接入层聚合原子化到具体用户场景下的接口和灵活性。 对微服务不熟悉的听众们/同行们没关系，在分享的最后我们会回到微服务，看看它和node.js的结合。

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598838731799.jpg)￼

```draft
  • 原子化服务 – 细粒度、独立部署、独立维护升级、独立扩容 
  • 所有服务内置平台层、应用层监控(Google Dapper类技术)
• 金管家、金钥匙、易淘金、开户系统。。。功能拆分、微服务容器、云化 
• 应用层“聚合” – 不同应用场景聚合不同的微服务 
可以看到我们的架构中
（性能速度损耗，但是用户不care，可能被传统的金融银行等『惯坏』了，关于钱财的还是别轻飘的好
```


### API Server 和 Node.js

好，现在我们知道了我们要在edge层实施一个强大的API 服务器来对接各种微服务，那在正式动手撩开袖子搞之前，有必要看下在更广的视野下理解它的历史和现状。因为在过于一段时间它们改变了不少。

最早的后端渲染页面，通过ajax来满足部分简单的前台交互（这时候后端MVC模型已经开始成熟 譬如rails, php,django 都是此中好手。

然后随着移动客户端iOS/Android快速发展和前端webapp化，越来越多的应用逻辑前移，富应用要求动态页面从而把渲染前移，所以此时接口要前后数据分离，所以restful这种基于资源为中心，加之http方法对应CRUD行动动作的，以status code 状态吗对应操作结果的接口框架和规范就流行起来。

但是问题还是有的：如资源接口的聚合上，接口数据的适用性上，要知道现在复杂的页面上不会那么傻傻的仅仅对应单个资源，它通常会依赖于多个相关资源的信息和部分信息。
所以有些基于RESTful扩展的接口约定协议来尝试解决。同时在新时代的，如GraphQL, Falcor，Meteor等来解决这些问题。


#### RESTful

那么什么是基于RESTful扩展呢。我们知道RESTful有它3层的成熟度模型，业界也有如Github， Heroku Platform API 提供的指南。对于 JSON API 我们需要在类似于opt-field筛选特定字段，嵌入关联资源等进行统一抽象的接口理解，来满足业务上的一些需要，最好是在http request 中间件层面就处理好，不需要业务的controller在parse这些urlparams和body。

```note
// 看起来每个开发者最后都会疑问那么关于API接口呢？很多人会直接想到RESTful API（因为太流行了），同时SOAP真的成为过去式了。同时现在也有不少其他标准如：HATEOAS, JSON API,HAL,GraphQL 等
// 第 2 级服务：使用多个 URI，不同的 URI 代表不同的资源，同时使用多个 HTTP 方法操作这些资源，例如使用 POST/GET/PUT/DELET 分别进行 CRUD 操作。这时候 HTTP 头和有效载荷都包含业务逻辑，例如 HTTP 方法对应 CRUD 操作，HTTP 状态码对应操作结果的状态。第3级服务： 使用超媒体hypermedia 作为应用状体引擎。
```


#### API Server 新方向

15年Facebook开源自己的relay，也引入了graphql（它是早在12年就被开始使用在它们的ios/andriod项目上）一种依赖于类型系统的数据查询拉取的描述语言。GraphQL 赋予客户端强大的能力（也是职责），允许它来实施几乎任意的查询接口。结合Relay，它能为你处理客户端状态和缓存，统一完成多个component的接口拉取。在服务器端实施GraphQL看起来比较困难。

网飞(NetFlix）的Falcor 看起来它也能提供那些Relay和GraphQL提供的功能，但是对于服务器端的实现要求很低。但现在它仅仅是开发者预览版没有正式发布。

这是现在流行的react技术栈（react+relay+graphql），我们发现之前ad-hoc query要多个复杂业务相关的接口需要后端实施，用graphql就简单多了，让client描述自己需要什么就行了。
![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598830486065.jpg)￼

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598832396579.jpg)￼


```note
// Servers publish a type system specific to their application, and GraphQL provides a unified language to query data within the constraints of that type system. That language allows product developers to express data requirements in a form natural to them: a declarative and hierarchal one.
```

#### Meteor

需要值得一提的是，Meteor，算是异类但是通过类似DDP，Remote Method，pub/sub等非常高效的完成了前后端的数据同步。想想看：
前端说模板绑定时需要绑定最新的用户feed，那么通过live query，每次数据库的内容变化，变化的内容会自动同步到前端，模板就会重新render。
通过remote method，你向后端调用数据就像是一个方法调用只不过不是进程间的是跨网络的。但是它太异类的，我们没法在现有的架构下使用它。

所以我觉得现在有什么方案完美的），所以我们还是需要结合我们的场景实现自己的API Server。这里我们就引进了koa2。

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14599935528704.jpg)￼




### Koa2 与 Node.js


- 它是什么
我们先首先来看koa2是什么. koa 我们都知道是由 Express 原班人马打造的，更小、更富有表现力、更健壮的 Web 框架。通过组合不同的 generator，可以免除重复繁琐的回调函数嵌套，并极大地提升错误处理的效率. 而koa2是通过es7的async/await 来进一步完善异步操作。关于express5和koa的历史可以看这篇文字.

- 何时发布
它还没有正式发布，官方会等到node.js实现了async/await后在正式发布。 
一般需要如下的步骤，等chromium来实现它，然后node.js把v8的代码合入并且做测试，一般6个月发布正式版本。 不过不用担心，官方已经说了很多人通过babel已经运用在自己的项目中了。而且微软最新edge的浏览器的类V8的ChakraCore js引擎，已经实现了它

- 哪些改动
它的一些具体改动也就是仅仅体现在中间件上，后续我们会具体看。


#### Why

那么为什么会选择它呢，主要是三个原因

应付异步IO我们有类似于callback，promise，node-fiber，generator/yeid的。

- 对于callback自然不用多说，把异步撸直了不用担心代码意大利面条式的。
- promise也是不错的异步原语但是比较verbose，then and then的。
- 比起generator更直白，需要wrap，co，yield，* 这些是什么鬼？！）

我们看下代码：这是读取目录下所有路径和markdown文件内容，然后拼接成字符串的操作。怎么样是不是非常直观不需要callback了，但是不用担心我们的操作仍然是异步的，不会阻塞和等待
await关键词必须在async function有有效，await通常等待一个promise的值。

```js
const fsp = require('fs-promise');

async function readDirContent(doc) {
  let paths = await fs.readdir('docs');

  let files = await paths.map(function(path){
    return fs.readFile('docs/' + path, 'utf8');
  });

  this.type = 'markdown';
  this.body = files.join('');
};
```


那错误处理呢，如果promise 抛异常可以被try catch住（）
之前express的错误处理相信大家也都知道同步错误可以在app.use的next error-handling middleware，但是对于异步代码中却无能无力因为在你进入回调中已经丢掉调用栈了。除非要在每个node.js惯例的error-first的callback中，手动处理或者把他next出去往上推。 
统一的错误处理意味着，就是说如同步代码的异常如json.parse对一个非法字符串进行转意时可以try catch，对于异步的类似于等待promise时被reject也能被try catch，但是如果你不去（否着会被吞掉这一点要除以）。所以我们一般需要类似这样的错误处理中间件放在全局的包下

```js
// error handler to JSON stringify errors
const errorRes = require('./middleware/error-res');
app.use(errorRes);

module.exports = async function(ctx, next) {
  try {
    await next();
  } catch (err) {
    if (err == null) {
      err = new Error('Null or undefined error');
    }
    // some errors will have .status
    // however this is not a guarantee
    ctx.status = err.status || 500;
    ctx.type = 'application/json';
    ctx.body = {
      success: false,
      message: err.stack
    };
    ctx.app.emit('error', err, this);
  }
};

// 用于关闭前的一些处理如保存数据，记录错误，发送邮件等等
process.on('uncaught', ()={});
```

中间件的写法也更直观了，这个是koa也都用的优势。看这个response-time的，在进入中间件时记住开始时间，然后await 等待后续中间件的执行，然后在结束后被交回执行权限后，计算diff
要知道得益于koa的回形针的写法，而不用像之前express那样，曲折

```js
function responseTime() {
  return async(ctx, next) => {
    var start = Date.now();
    await next();
    var delta = Math.ceil(Date.now() - start);
    ctx.set('X-Response-Time', delta + 'ms');
  }
}
```

#### 我们的koa中间件

  - compose koa
  - 常见的middleware
  - koa-adapter （常见中间件的引入）
  - 文件即路由
  - koa-validator

中间件是非常重要的概念。要知道，一个Koa的应用就是包含一组async函数写的中间件的对象，然后按照一定顺序对请求操作返回响应。

##### koa composite

那么我们看多个中间件是怎么运行的。我们发现请求先后从上往下进入进入response-time, logger, content-length, body 函数中，在body函数中我们执行yield后面的设置我们body内容后因为是最后一个中间件，所以执行又继续从下往上执行之前中间件yield后面的部分如设置header头如content-length，response-time, 打log等。整个执行顺序非常像右侧图表现的回形针的样子。

![](http://gaohailang.github.io/node-party-gf-security-practice/images/koa-middleware.gif)￼￼


那么具体是怎么实现的呢？koa-compose 就是内部实现。下面这是它具体的代码（感兴趣的可以看下，可以发现就这十几行的代码就把这些中间件串联起来的逻辑

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598798348020.jpg)￼

这些是比较常见的中间件，官方wiki上有详细的成熟的列表。如cookie的，body parse的，等等

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598798901987.jpg)￼


##### koa adapter

把es6的generator和yield的变成es7的async/await写法
既然我们用上来koa2，那之前的koa1的中间件怎么办呢，官方推荐做法是在代码仓库提供next分支，来host新的async版本的中间件。当然可以通过koa adapter 这样的中间件来替我们转换。

```js
// use Koa 1.0 middleware
app.use(function*(next) {
  const start = Date.now()

  yield next

  const ms = Date.now() - start
  console.log(`${this.method} ${this.url} - ${ms}ms`)
})

// koa-logger@1 only support koa@1
const logger = require("koa-logger")

// use legacy middlewares with adapt(...)
app.use(adapt(logger))
```

```note
// 内部实现co.wrap。 If you want to convert a co-generator-function into a regular function that returns a promise, you now use co.wrap(fn*).
```

##### Valiate

关于验证我们有两个，请求入参如query,params,body的检查如是否存在，格式如email，字符长度等

关于数据模型的验证，mongodb 本身是schema-less，但这并不意味着我们容忍脏数据的随意插入（只是方便我们修改和扩展数据Schema，方便业务发展）。hapi的joi提供了很好的API，mongoose也有自己的plugin机制来实现validate，社区也有插件来统一把两套schema统一如从joi的生成mongoose的schema。

```js
// 关于请求的入参验证
ctx.checkQuery('query', 'Invalid query').notEmpty();
ctx.checkQuery('type', 'Invalid type').
    isIn(baseSearchTypes.concat(['all', 'stock']));
    
function assertPagintionQuery(ctx) {
  ctx.checkQuery('page', 'Invalid page').notEmpty().isInt();
  ctx.checkQuery('size', 'Invalid size').notEmpty().isInt();
}

var joiUserSchema = Joi.object({
     name: Joi.object({
         first: Joi.string().required(),
         last: Joi.string().required()
     }),
     email: Joi.string().email().required(),
     bestFriend: Joi.string().meta({ type: 'ObjectId', ref: 'User' }),
     metaInfo: Joi.any()
 });
```

### 日常开发 与 Node.js

以上是koa web相关的，当然我们日常开发也少不了其他部分。譬如现在代码ES6化，构建npm scripts化（它会集中在持续集成，代码质量上），上线前准备（如性能调优，安全），web开发上下游等

- es6
- npm scripts
- 性能调优
- 上线准备
- 安全

#### ES6化
##### 逐渐迁移legacy 代码

很多旧代码用es5写起来比较verbose，可以使用最新的es6语法来改造，精简轻量很多。如解析构，模板字符串等等都不错。如之前从req.query中去数据。现在xxx

```js
// before
var page = req.query.page,
  , size = req.query.size;
// after
let {pgae, size} = req.query;
rp({
  uri, params: {page, size}
});
```

##### Babel 集成

之前有些人吐槽说babel改过后，代码xxx，反正我们是没有遇到，代码不够复杂？！
首先确保你的基础node版本不小于5.6，Tip: 使用node 5确保我们的babel transpile 可以尽量让产出的代码精简 。因为大部分新的es6的语法在Node5中已经被实现了不需要babel再去transpile（听说有转错的风险还有性能也不好)

你会发现使用 similiairty 去跑没啥差别，我们看下 .babelrc 中的preset为 es2015-node5，然后就有两个babel的插件用于转换2a的代码

部署es6代码用于线上生产（先构建好 es5-compatible，加入到docker镜像中 ）
eslint


```json
{
  "presets": ["es2015-node5"],
  "plugins": [
    "transform-async-to-generator",
    "syntax-async-functions"
  ]
}
```

#### npm scripts 构建过程

现在的一大趋势，是把多余的Gulp也好，Grunt也好，去除掉。
为什么？因为npm本身提供很好的脚本支持，它不需要都与的gulp wrap(因为你还需要依赖于它去包少了或者它出bug都是问题），直接引入了你需要的工具如（uglifyjs, cssmin, babel等），通过灵活的hook来做一些构建task的设置

那我们看看用它可以具体做什么事情和我们又是怎么做到的

##### 及时更新的依赖

想必大家对前不久的left-padding的事件都有耳闻。一位开发者下架了自己的仅仅用于格式化字符串的一个函数就导致了很多开源项目构建失败。所以我们给出的建议是通过shrinkwrap锁定住要上线的版本，同时定期的通过npm check来检查依赖组件的更新情况。

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598767264510.jpg)￼

##### file watch

这个特性在前端开发中已经习以为常了（如livereload, browsersync）。我们当然也可以通过pm2来设置，看个人喜好。为他配置一些需要ignore的目录，然后自由编码去吧。
运行 npm run dev就可以了， 可以看到ns中的一些特殊变量如 $npm_package_main来指定babel-node这个解释器来运行我们的入口文件（这样在开发时不需要编译代码

```json
{
  "main": "index.js",
  "scripts": {
    "dev": "nodemon --exec babel-node -- $npm_package_main"
  }
}
```


```note
nodemon  Simple monitor script for use during development of a node.js app.
```

##### 入库检查 

- npm run precommit : npm test
- npm run prepush : npm-run-all lint test test:deps
- npm run inspect : jsinspect

使用 `hurky` 可以修改你的git命令，提供hook点在commit/push/merge 前执行检查。
譬如我们利用npm的hook（pre)在提交commit前运行下我们的单元测试等（在那些），在推送代码仓库前，执行我们的lint检查是否良好的代码格式等等。
甚至我们可以运行inspect，看看我们是否有存在代码的copy&paste这种情况。

说到测试，很多人说项目很赶没时间啊。还有人有些测试用例写起来还naive，不想写。其实我们并不需要对所有代码做测试。尤其在迭代速度很块的情况下很多需求没理清楚，说不定一些前天的在明天就要remove掉。
那么我们会集中在如下部分：

对这些进行测试：

- 那些经常要被修改的 change a lot 
- 那些有很高复杂度的 risky
- 那些重要功能/常被查看的 more traffic

##### 代码质量检查

我们真的应该非常关注我们的代码质量，想想看对于关键代码如基础组件，核心业务功能如果维护性不好，后续的需求和代码的更改都很困难等。
因为我们也接手过（相信大家也是），我们能不能避免这些呢，不要给自己挖坑（事实上我们也乐于给自己挖坑哈 - 啪啪啪打脸程序员的九本指南）以不符合设计原理 / 不易维护 / 不易调整 / 不够健壮 / 不够美观的方式解决问题
我们可以通过plato，从 代码复杂度， 代码行数（维持在100一下），lint出的错误数等，去关注它。
它也提供的基于日期对关键指标的统计看看代码改进的趋势，是不是朝着好的方向还是走想可怕的不好维护急需重构的深渊。

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598775638257.jpg)￼

确保自己不要变成一下说的情况：
![94419d84gw1f2jqooa7i9j213y10an94](http://gaohailang.github.io/node-party-gf-security-practice/images/94419d84gw1f2jqooa7i9j213y10an94.jpg)￼￼


#### JS Smells 代码坏味道

我之前也从国外的slide整理过一篇文章，对于代码坏味道进行了回顾。感兴趣的可以多看看，看看那些有问题的代码是不是和我们的很像，o(╯□╰)o。

我们当然可以通过 eslint rule 来发现：
如不允许复杂的switch语句，不允许重复reassign等，代码复杂度（if/else等嵌套不超过5等）

哈哈，希望我们尽量不写出这种糟糕的代码：

![](http://gaohailang.github.io/node-party-gf-security-practice/images/54aeed78b5e8033c6b14.png)￼

#### 在线上运行（Production Deploy & Security）

当然咯，我们代码通过层层考验，最终准备上线了，最好还是需要一些确保如对性能进行调优，具体koa上线设置，安全上有哪些考虑等。

##### 性能调优

我强烈推荐你们看strongloop（他从TJ手中取得Express的项目权限，接着被IBM收购专注于Node.js企业开发）出品的系列博文，看看如何优化。

[也来八卦：Express 的奇葩发展史](https://github.com/gaohailang/blog/issues/7)

- [heap profiling](https://strongloop.com/strongblog/node-js-performance-heap-profiling-tip/)
- [memory leak diagnosis](https://strongloop.com/strongblog/node-js-performance-tip-of-the-week-memory-leak-diagnosis/)
- [CPU profiling](https://strongloop.com/strongblog/node-js-performance-tip-cpu-profiler/)
- [scaling proxies clusters](https://strongloop.com/strongblog/node-js-performance-scaling-proxies-clusters/)
- [event loop monitoring](https://strongloop.com/strongblog/node-js-performance-event-loop-monitoring/)
- [garbage collection](https://strongloop.com/strongblog/node-js-performance-garbage-collection/)


- 通过heap profiling 看我们的内存使用是否合理，诸如对象，Request，String，Timer都占用了多少
- 通过内存泄露的诊断，看看我们是不是有一些没有回收的导致内存超量使用（1G左右)，快速泄露是非常容易诊断的，那么那些慢速的呢通常运行一天才会挂掉的呢
- 通过CPU profiling，我们看看那些同步代码消耗了计算资源
- 通过clusters，pm2 已经内建支持了不需要cluster模块引入，来在多核上同时绑定和serve同一个端口的请求提升应用的性能
- node.js 非常关键的event loop，在这种单线程下NIO非阻塞IO下实现高并发模型。但是前提是你的代码不要阻塞它，导致后续的请求不能及时被serve。
- 最后是garbage collection，这篇文字可以看下v8是怎么管理内存的，heap被分成哪些不同空间。这对于想要精细控制gc非常有帮助。

##### Best Practices in Production 

- Gzip压缩中间件/Nginx开启
- 静态文件中间件/Nginx加入
- 避免尽量少的复杂同步代码（Event Loop 监控）
- 打好全量的log信息（通过环境变量和进程信号调整）
- 合理的处理异常

如果你的应用很轻不需要前置的nginx，那么需要在引入诸如gzip, static service 这些中间件来高效处理。或者如果你的应用如果有前面的反向代理的nginx，那么开启它们。
正如之前提到的，不要使用复杂的同步代码阻塞你的event loop。要打好全量的log来监控你应用发生的一切即包括了operation error也要含有system error等。还可以用类似debug模块调整log显示的级别如在线上问题诊断开启verbose，没问题后再把环境变量改成info，warning之类来通过发送进程信号量给应用调整。
PS在我们一些关键的金融理财业务很多log都是不允许的删除的为了审核和合规，所以做好rotate等
这方面很多公司都非常了积累了，最后又一些[链接](https://strongloop.com/strongblog/best-practices-for-express-in-production-part-two-performance-and-reliability/)


##### 安全施工

- 不要使用过期老旧的版本框架
- 快点用上 https
- Helmet 中间件
- 安全使用Cookie & Session
- 确保其他组件依赖是安全的
- 其他一些基础的：[security-checklist](https://blog.risingstack.com/node-js-security-checklist)


#### 其他（[Node] - 16年，新 Node 项目注意点）

  我们之前也分享过16年新node项目有哪些注意点的文章，这里也简单提下。
  
  版本管理刚才有提到，需要特别重要npm install要加上--save， save-dep的选项不然会漏掉，或者你可以通过npmrc来设置默认就保存等
  
  web应用开发的十二条军规是来自于rails社区关于web开发的一些通用型原则很有名很值得一读。
  还有诸如代码风格结合eslint的统一等等
 
 
### Web 开发 和 Node.js

- 关于openresty & haproxy & varnish
- 关于 mongodb, redis
- 关于log，监控和指标计算
- 关于扩容，kubernetes
- 关于 kakfa 等

Web开发不仅仅包括了应用Server逻辑本身的书写，我们还需要关注上下游的组件。

- 最前面的接入，可能需要openresty实现我们内嵌在nginx中的一些业务逻辑非常高性能（做一些风控和，session管理啊还是非常有效的）。锤子就把去年的门票收入捐给他们呢。
- 还有HAProxy做内部负载均衡，varnish做一些接口的缓存和精细控制
- 常见的存储用使用kv的redis热数据，mongodb文档型数据库来落地业务数据等非常不错在我们的场景下
- 关于log，一些关键的业务事件需要通过fluentd，flume等发送给大数据，还有一些要通过storm/spark流式计算来实施我们的内部数据的Dashboard和监控等
- kafka利用它高吞出的性能做在我们业务里承担一些消息队列的角色
- 我们的扩缩容怎么做，docker容器需要管理，用k8s。

那么最终还是要回到我们应用的运行环境， docker容器和k8s中，看看微服务到底是。

### 微服务和Node.js

#### 什么是微服务

这种架构方式并没有非常准确的定义，但是在业务能力、自动部署、端对端的整合、对语言及数据的分散控制上，却有着显著特征。
微服务架构风格，就像是把小的服务开发成单一应用的形式，每个应用运行在单一的进程中，并使用如HTTP这样子的轻量级的API。这些服务满足某需求，并使用自动化部署工具进行独立发布。这些服务可以使用不同的开发语言以及不同数据存储技术，并保持最低限制的集中式管理。

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598813696713.jpg)￼

这是它的大概组成，在左下角是入口的api gateway也是我们node重点关注的地方，在具体微服务实现后专门被路由到需要服务发现的机制，单独的微服务需要怎么被部署要借助于容器等新的运维devops的支持。[具体文章](https://github.com/gf-rd/blog/issues/10)


```note
需要在一些节点加上agent，实现动态扩容等(mind shift from 机器性能到集群扩展容，随时挂掉预设）
公司整理的一些规范：docker化，无状态。从而方便Kubernets扩缩容或调整机器资源（重启等），或者有状态恢复机制，环境变量配置优先（配置项通过env传入，如数据库，redis等，不要使用卷映射（如logs文件通常打到console中，然后fluentd传入大数据（走kafka; node_modules ADD file 到docker中，提供健康检查脚本
```

#### 微服务集成开发

那么我们想实践微服务，node.js web上需要做哪些工作了？

- docker 部署
- 无状态（随时被动态扩缩容掉
- request监控（链路追踪
- 配置加载管理（环境变量

##### Docker 部署

Docker, 用来打包，分发和在容器中运行应用的好用工具。它和虚拟机是类似的，独立应用和它的依赖到独立自包含的直到你可以在其他地方运行、允许我们更有有效地使用计算资源、他们的主要差别体现在它们的架构实现方案上

这是它主要部分和命令的图片，感兴趣的又可以看我的翻译的文章[容器，Dokcer，虚拟化 - 写给开发者的入门指南](https://github.com/gaohailang/blog/issues/13)

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598810519526.jpg)￼

这是关于docker操作相关的命令，我们集成在npm scripts 中了

```json
{
  "rsync:gftest": "rsync --cvs-exclude -cauvz -e \"ssh -A gf@<relay-ip> ssh\" .es5 ubuntu@<dest-ip>:/opt/gf/gfwealth-composite/",
  "docker:prebuild": "rsync -avz --exclude .es5 --exclude .idea . ./.es5 && babel . --out-dir ./.es5 --ignore ./.es5,./node_modules",
  "docker:build": "cd .. && docker build -t <private-docker-host>/gfwealth-composite:koa-2016-03-17 .",
  "docker:run": "sudo docker run -d --name gfwealth-composite -p 8000:9321 -v /opt/gf/gfwealth-composite/logs:/opt/gf/gfwealth-composite/logs -v /etc/localtime:/etc/localtime:ro --env-file=\"config/env/product\" docker.gf.com.cn/gfwealth-composite:koa-2016-03-17",
  "docker:restart": "npm run docker:run && npm run docker:rm"
  "docker:rm": "sudo docker rm -f gfwealth-composite",
  "docker:push": "docker login docker.gf.com.cn && docker push <private-docker-host>/gfwealth-composite:koa-2016-03-17"
}
```

##### request: 监控和debug

我们的web应用接受请求，也发送请求。 我们需要对这些请求进行记录和追踪，这样才能对一些用户的具体操作进行trakc甚至replay来辅助我们debug用户的线上问题。
尤其在我们微服务下，对于大量原子化的外部微服务的请求返回数据进行审视和使用杜绝可能存在的外部broken或者不符合预期的返回搞挂我们的应用，辅助和对口同事撕逼。

这是我们的代码，request本身支持debug模式，我们通过request-debug来对请求响应生命周期中的关键点进行监听，记录log。如状态码，响应时间，用户id，返回的部分结果等等。

```js
require('request-debug')(rp, function(type, data, r) {
  // put your request or response handling logic here
  // Todo: request-post data lost? response json.stringify broken?
  if(type === 'request') {
    var {debugId, uri, method, body} = data;
    var userId = data.headers.userId;
    var more = body ? '' : ('-'+body) + userId ? '' : ('userId:'+userId);
    debug(`${debugId}-${method}-${uri}` + more);
  }

  if(type === 'response') {
    var {debugId, statusCode, body} = data;
    debug(`${debugId}-response-${statusCode}-`+JSON.stringify(body).slice(0, 155));
  }
});
```

我们当然可以通过一些工具来可视化这些，如下面RisingStack的strack和Google开源的zipkin，这个分布式的追踪系统来帮忙做这事情，如清晰看到整个请求链路中依赖的接口个别耗时等。

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598809054912.jpg)￼

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598809442901.jpg)￼


##### 配置加载&环境变量

我们使用paypal的confit，来加载默认的配置，还会根据通过环境变量env来自动加载环境相关配置文件（做到差异化配置，还有很多增强协议如从环境变量中读取，从yaml文件，从特定文件读取内容，读取相对目录路径到绝对路径等等。
从环境变量读取你的外部依赖非常重要在微服务下面，我们依赖于此
社区还有nconf，选一个你顺手的使用起来吧。

- 各种 protocol handler 
- 基于不同环境实现差异化配置
- 根据env变量自动加载

```js
// envfile - 可以在 docker run 的时候以env-file 参数带入
GFWC_storeExShopList=http://shopdev.gf.com.cn/api/store/shop/excellentshop/{id}/{page}/{size}

// 配置：
{
  "env": "dev",
  "api": {
    "articleSearch": "env:GFWC_articleSearch"
  }
}
```

### 我们和开源

我们非常重视开源技术，去IOE化。
所以从去年开始，开源上的做了一些工作，回馈社区：

- 我们团队的[汤老师](https://github.com/lightningtgc/)在去年去QCon上海在将我们在es6上的一些前沿实践，业界也有我们[es6 coding style](https://github.com/gf-rd/es6-coding-style)，可以参考。
- 同时去去年尾声开始在angular2上写书准备，翻译了官方的guide，所以我们收到Google官方的邀请让我们推动ng2在国内文档化的工作。
- 更多的一些技术分享，可以在我们的[Github 的技术博客](https://github.com/gf-rd)中看到

可以看到正是我们在技术上的尝试，使得我们在 Node 及其相关的技术选型形成来今天的风格

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598823363661.jpg)

### 结束

#### 回顾下

今天很大而全的跟朋友们过了下广发在一些技术上的实践，思考和观点。每个知识点一般都会有外链给感兴趣的同学会后继续学习和了解。

- 先讲下我们技术实践产生的背景 - 即我们这个团队是谁
- 然后是我们今天的主角node.js的在团队内的选型定位
- 我们怎么看待要实施的API Server，看它的发展历史和未来展望
- 接着我会先后从 koa2相关的实践，普通的node开发进行讲诉，最后以微服务结尾。

后端的纵深很多，前端的涉及面很宽，这就是我们折腾的现状。

#### 加入这场 FinTech Storm

谢谢大家，可能今天的内容比较多，不管怎么样希望今天的分享能让大家有些收获。
我个人的联系方式是：[ghlndsl](https://raw.githubusercontent.com/gaohailang/blog/master/source/images/wechat-qrcode.jpg)，[微博](http://weibo.com/1siva)，[Twitter](https://twitter.com/ghlndsl)），邮箱：gaohailang@gf.com.cn

我们现在还持续长期的招人，准备好FinTech Storm了，来撩开袖子一起和我们 `玩到想玩的技术，赚到想赚的钱` ~

没过瘾，来深圳海岸城约饭交流，SCC 8 楼16年中我们会有分享会，更深的技术交流还有金融业务知识

![](http://gaohailang.github.io/node-party-gf-security-practice/images/14598902035581.jpg)
