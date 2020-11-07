

## 不可错过的Webpack核心知识点

双鱼 前端瓶子君 *10月23日*

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQQPKIGLZ0SchqbdpjWaQYibkA761MIkTICzicqQXmUaibZiaFxPjjeKESLkTpOPXbSm3rsHcAKFJQTC8Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

本文使用「署名 4.0 国际 (CC BY 4.0)」 许可协议，欢迎转载、或重新修改使用，但需要注明来源。

> 作者: 百应前端团队 @双鱼
>
> https://juejin.im/user/307518985745895

## 1. 核心概念

- entry：入口。webpack是基于模块的，使用webpack首先需要指定模块解析入口(entry)，webpack从入口开始根据模块间依赖关系递归解析和处理所有资源文件。
- output：输出。源代码经过webpack处理之后的最终产物。
- loader：模块转换器。本质就是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果。因为 Webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作。
- plugin：扩展插件。基于事件流框架 `Tapable`，插件可以扩展 Webpack 的功能，在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。
- module：模块。除了js范畴内的es module、commonJs、AMD等，css @import、url(...)、图片、字体等在webpack中都被视为模块。

另外webpack4开始 mode 变成一个重要概念，webpack为不同 mode提供了一些默认值，附上阮一峰老师的吐槽

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

不同mode的默认配置如下：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 2. 打包流程

1. 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
2. 初始化编译：用上一步得到的参数初始化 Compiler 对象，注册插件并传入 Compiler 实例（挂载了众多webpack事件api供插件调用）；
3. AST & 依赖图：从入口文件（entry）出发，调用AST引擎(acorn)生成抽象语法树AST，根据AST构建模块的所有依赖；
4. 递归编译模块：调用所有配置的 Loader 对模块进行编译；
5. 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
6. 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统；

> 在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到相关事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果

##### 构建流程核心概念：

- Tapable：一个基于发布订阅的事件流工具类，Compiler 和 Compilation 对象都继承于 Tapable
- Compiler：webpack编译贯穿始终的核心对象，在编译初始化阶段被创建的全局单例，包含完整配置信息、loaders、plugins以及各种工具方法
- Compilation：代表一次 webpack 构建和生成编译资源的的过程，在watch模式下每一次文件变更触发的重新编译都会生成新的 Compilation 对象，包含了当前编译的模块 module, 编译生成的资源，变化的文件, 依赖的状态等

更加细化的构建流程图：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

看大图点这里👈

> 流程图出处：淘系前端团队-细说 webpack 之流程篇

## 3. Loader

loader就像一个翻译官，将源文件经过转换后生成目标文件并交由下一流程处理

##### 使用方法

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

- 每个loader职责都是单一的，就像流水线上的工人
- 顺序很关键（从右往左）

##### 实现准则

- 简单【Simple】loader只做单一任务，多个loader > 一个多功能loader
- 链式【Chaining】遵循链式调用原则
- 无状态【Stateless】即函数式里的Pure Function，无副作用
- 使用工具库【Loader Utilities】充分利用 loader-utils 包

##### 实现一个简单的loader，功能是替换console.log、去除换行符、在文件结尾处增加一行自定义内容

```
/** webpack.config.js */  
  
const path = require("path");  
  
module.exports = {  
  entry: {  
    index: path.resolve(__dirname, "src/index.js"),  
  },  
  output: {  
    path: path.resolve(__dirname, "dist"),  
  },  
  module: {  
    rules: [  
      {  
        test: /\.js$/,  
        use: [  
          {  
            loader: path.resolve("lib/loader/loader1.js"),  
            options: {  
              message: "this is a message",  
            }  
          }  
        ],  
      },  
    ],  
  },  
};  
/** lib/loader/loader1.js */  
  
const loaderUtils = require('loader-utils');  
  
/** 过滤console.log和换行符 */  
module.exports = function (source) {  
  
  // 获取loader配置项  
  const options = loaderUtils.getOptions(this);  
  
  console.log('loader配置项:', options);  
  
  const result = source  
    .replace(/console.log\(.*\);?/g, "")  
    .replace(/\n/g, "")  
    .concat(`console.log("${options.message || '没有配置项'}");`);  
  
  return result;  
};  
  
```

##### 异步loader如何编写

```
/** lib/loader/loader1.js */  
  
/** 异步loader */  
module.exports = function (source) {  
  
  let count = 1;  
  
  // 1.调用this.async() 告诉webpack这是一个异步loader，需要等待 asyncCallback 回调之后再进行下一个loader处理  
  // 2.this.async 返回异步回调，调用表示异步loader处理结束  
  const asyncCallback = this.async();  
  
  const timer = setInterval(() => {  
    console.log(`时间已经过去${count++}秒`);  
  }, 1000);  
  
  // 异步操作  
  setTimeout(() => {  
    clearInterval(timer);  
    asyncCallback(null, source);  
  }, 3200);  
  
};  
  
  
```

## 4. Plugin

在webpack编译整个生命周期的特定节点执行特定功能

##### 实现要点：

- 一个命名JS函数或者JS类
- 在prototype上定义一个apply方法（供webpack调用，并且在调用时注入 compiler 对象）
- 在 apply 函数中需要有通过 compiler 对象挂载的 webpack 事件钩子（钩子函数中能拿到当前编译的 compilation 对象）
- 处理 webpack 内部实例的特定数据
- 功能完成后调用 webpack 提供的回调

##### 基本模型：

```
// 1、Plugin名称  
const MY_PLUGIN_NAME = "MyBasicPlugin";  
  
class MyBasicPlugin {  
  // 2、在构造函数中获取插件配置项  
  constructor(option) {  
    this.option = option;  
  }  
  
  // 3、在原型对象上定义一个apply函数供webpack调用  
  apply(compiler) {  
    // 4、注册webpack事件监听函数  
    compiler.hooks.emit.tapAsync(  
      MY_PLUGIN_NAME,  
      (compilation, asyncCallback) => {  
  
        // 5、操作Or改变compilation内部数据  
        console.log(compilation);        
  
        console.log("当前阶段 ======> 编译完成，即将输出到output目录");  
  
        // 6、如果是异步钩子，结束后需要执行异步回调  
        asyncCallback();  
      }  
    );  
  }  
}  
  
// 7、模块导出  
module.exports = MyBasicPlugin;  
```

##### 实现一个plugin，功能是在dist目录自动生成README文件：

```
const MY_PLUGIN_NAME = "MyReadMePlugin";  
  
// 插件功能：自动生成README文件，标题取自插件option  
class MyReadMePlugin {  
  
  constructor(option) {  
    this.option = option || {};  
  }  
  
  apply(compiler) {  
    compiler.hooks.emit.tapAsync(  
      MY_PLUGIN_NAME,  
      (compilation, asyncCallback) => {  
        compilation.assets["README.md"] = {  
          // 文件内容  
          source: () => {  
            return `# ${this.option.title || '默认标题'}`;  
          },  
          // 文件大小  
          size: () => 30,  
        };  
        asyncCallback();  
      }  
    );  
  }  
}  
  
// 7、模块导出  
module.exports = MyReadMePlugin;  
  
```

`compiler.hooks` 上挂载了不同时期触发的webpack事件函数（类似于React生命周期），可以在编译的各个阶段执行其他逻辑或者改变输出结果，具体支持的事件列表可以看这里：compiler-hooks

##### Tapable：

webpack 的插件架构主要基于 Tapable 实现的，Tapable 是 webpack 项目组的一个内部库，主要是抽象了一套插件机制。它类似于 NodeJS 的 EventEmitter 类，专注于自定义事件的触发和操作。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Tapable事件类型分为同步和异步，内部又以不同的规则分为不同类型，上述事件的具体区别可以看 这篇文章，理解这些事件的区别和应用场景有助于我们理解webpack源码和编写Plugin

##### Complier对象：

在webpack启动时被初始化一次，全局唯一，可以理解为webpack编译实例，它包含了webpack原始配置、Loader、Plugin引用、各种钩子

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

部分源码：https://github.com/webpack/webpack/blob/10282ea20648b465caec6448849f24fc34e1ba3e/lib/webpack.js

## 5. 性能优化

##### 1. 从何开始？

- 使用 speed-measure-webpack-plugin 测量打包速度

  ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

- 使用 webpack-bundle-analyzer 进行体积分析

  ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  从某项目的分析图可以看出一个很明显的优化空间就是 BizCharts 没有按需引入，这时候我们可以import路径再执行一次打包分析看效果。

  另外图中每个模块都有三种Size，分别是 Stat Size、Parsed Size、Gzipped Size，这三者的分别代表什么含义可以看下插件的github issue

##### 2. 优化Loader配置

思路主要是优化搜索时间、缩小文件搜索范围、减少不必要的编译工作，具体做法可以看以下配置文件

```
module .exports = {   
  module : {   
    rules : [{  
      // 如果项目源码中只有 文件，就不要写成/\jsx?$/，以提升正则表达式的性能  
      test: /\.js$/,   
      // babel-loader 支持缓存转换出的结果，通过 cacheDirectory 选项开启  
      use: ['babel-loader?cacheDirectory'] ,   
      // 只对项目根目录下 src 目录中的文件采用 babel-loader  
      include: path.resolve(__dirname,'src'),  
      // 使用resolve.alias把原导入路径映射成一个新的导入路径，减少耗时的递归解析操作  
      alias: {  
        'react': path.resolve( __dirname ,'./node_modules/react/dist/react.min.js'),  
      },  
      // 让 Webpack 忽略对部分没采用模块化的文件的递归解析处理  
      noParse: '/jquery|lodash/',  
    }],  
  }  
}  
```

##### 3. DLL Plugin Or Externals

合理使用DLLPlugin将更改频率较低的代码（三方库）移到单独的编译中，我理解大部分场景下和配置 externals 作用是差不多的（都不用打包三方库），但是 externals 在某些场景下会存在失效问题，具体可以看 这篇文章，另外 DLLPlugin 具体使用 参考这里

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

##### 4. 多进程系列

多进程阵营里有几位知名选手：

- thread-loader（v4以后的官方推荐）
- happypack（不怎么维护了）
- parallel-webpack（不怎么维护了）

这里只介绍一下 `thread-loader` ，使用 `thread-loader` 将开销较大的 loader（例如babel-loader）放到独立进程中（官方描述 worker pool）处理，使用上有以下注意事项

- 将其放在需要单独加载的loader的前面，顺序很关键

```
module.exports = {  
  module: {  
    rules: [  
      {  
        test: /\.js$/,  
        include: path.resolve("src"),  
        use: [  
          "thread-loader",  
          // your expensive loader (e.g babel-loader)  
        ]  
      }  
    ]  
  }  
}  
```

- worker pool中的loader使用上是有限制的，例如无法使用自定义 loader api，无法获取webpack 配置项

##### 5. 合理利用缓存 缩短非首次构建时间

目前项目在用的插件是 `hard-source-webpack-plugin`，效果较为显著，不过缺点有3

1. 生成的缓存文件较大，比较占用磁盘空间（之前还出现过发布的时候误把缓存文件上传到服务器导致发布特别慢的情况 =。=，所以最好还是指定缓存文件路径为 node_modules 内部）
2. 这个仓库也很久没更新了
3. 现有项目偶尔会出现更改代码不触发重新编译的情况，猜测可能与此插件有关

另外 webpack5 是否有自带的缓存策略或者官方维护的缓存插件还需要去了解一下

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

##### 6. 代码压缩 减少产物体积

- webpack3配置optimization.minimize = true会默认启用 UglifyJsPlugin，其多进程版本为 ParallelUglifyPlugin
- webpack4 中 webpack.optimize.UglifyJsPlugin 已被废弃，默认内置使用 terser-webpack-plugin 插件压缩优化代码，原生支持多进程（这里想起官方文档 Build Performance 章节中列举的优化措施第一点：Stay Up to Date，最香的还是最新的webpack版本）

##### 7. Code Splitting

官方文档描述的code splitting的3种姿势：

1. 多entry配置（多entry是天然的code splitting，但是基本没人会因为性能优化的点去把一个单页应用改成多entry）

2. 使用 SplitChunksPlugin 进行重复数据删除和提取

   ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

3. 使用 Dynamic Import 指定模块拆分，并且可以结合 preload、prefetch做更多用户体验上的优化

   ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 6. 想的更远：那些值得深究的问题🤔

- HMR的原理？
- Tree shaking原理，为什么需要es module的写法？
- webpack5的Module Federation有哪些优势，在与http2.0的结合上有哪些有趣的事情，在微前端上的应用？
- 为什么说rollup比webpack更适合打包组件库？

