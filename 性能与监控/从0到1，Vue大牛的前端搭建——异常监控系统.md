## 从0到1，Vue大牛的前端搭建——异常监控系统

开课吧然叔 前端瓶子君 *8月31日*

来源： 开课吧前端团队

[https://mp.weixin.qq.com/s/4UyEHM-YmdgrfF_yze9Bpg](https://mp.weixin.qq.com/s?__biz=MzUzMjA3MTI2NQ==&mid=2247485042&idx=1&sn=f957ad6e31a4f6ffddaba91e1036da38&scene=21#wechat_redirect)

**本篇文章读后，你将GET的技能：**

●收集前端错误（原生、React、Vue）

●编写错误上报逻辑

●利用Egg.js编写一个错误日志采集服务

●编写webpack插件自动上传sourcemap

●利用sourcemap还原压缩代码源码位置

●利用Jest进行单元测试

有没有心动的感觉？赶紧跟然叔学起来吧！

## 一、如何捕获异常

**JS异常：**js异常的特点是,出现不会导致JS引擎崩溃 最多只会终止当前执行的任务。
比如一个页面有两个按钮，如果点击按钮发生异常页面，这个时候页面不会崩溃。
只是这个按钮的功能失效，其他按钮还会有效☟

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSh7yth48gDEhzib2DSeV2ibLcVv4vNHDt9V0B4icBrXhewQNemUMznBt1eQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSh0DTWw5eL3r5ASdubSwKgdR4qEn5bc2SL88icny1J4O5OmhbohcGszhw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上面的例子我们用setTimeout分别启动了两个任务。
**虽然第一个任务执行了一个错误的方法。程序执行停止了。但是另外一个任务并没有收到影响。**
其实如果你不打开控制台都看不到发生了错误。好像是错误是在静默中发生的。
下面我们来看看这样的错误该如何收集。

**try-catch：**
JS作为一门高级语言我们首先想到的使用try-catch来收集。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShJetDIoMg0drAWo9fLZicVzmgAvEfiaPQLCTPILXDkvprFepibSKM4Nk5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSh3IaWM3Gicyfass06t1YqxQtBqUbFvm78YTPkK1aBMYb5Y2dq3p6aGog/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果在函数中错误没有被捕获，错误会上抛。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShXibEDPYhyBux8whHN097h7ksNLtJo2N0ZneD6ZOgPG3dUN3ffVgvteg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSh4MRbN16ibctB1Lnw60QEmiabyfQu8tibkIomZZuczVOYprHoDQ80kjOJA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

控制台中打印出的分别是错误信息和错误堆栈。
读到这里大家可能会想那就在最底层做一个错误try-catch不就好了吗。
确实作为一个从java转过来的程序员也是这么想的。
但是理想很丰满，现实很骨感。我们看看下一个例子。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShT8phzfn8qhLbma1oUN44vDxrzTKNOc2KEWSqDkuYMKgohg4icbf8jhg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShVGOLLSibNo2zBicMZn3SEHmV8JzRWudic2EOVfxUo863FOPluAMb2FPxA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

大家注意运行结果，异常并没有被捕获。
这是因为JS的try-catch功能非常有限一遇到异步就不好用了。
那总不能为了收集错误给所有的异步都加一个try-catch吧，太坑爹了。
其实你想想异步任务其实也不是由代码形式上的上层调用的就比如本例中的settimeout。
大家想想eventloop就明白啦，其实这些一步函数都是就好比一群没娘的孩子出了错误找不到家大人。
当然我也想过一些黑魔法来处理这个问题比如代理执行或者用过的异步方法。
算了还是还是再看看吧。

## 二、异常任务捕获

**window.onerror:**

window.onerror 最大的好处就是可以同步任务还是异步任务都可捕获。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSh2qRNklLzsTn9aS7zRvH01QdqOlVHapMyCKZAicTJomENRLg850GmDhw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShyMsicgqYKWf4KTLZrDiaibaCLCwgBjMCjeeWD3Gc6vS3YoCBEuvSAwUzg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

onerror返回值
onerror还有一个问题大家要注意 如果返回返回true 就不会被上抛了。
不然控制台中还会看到错误日志。

监听error事件:
文件中的位置☟window.addEventListener('error',() => {}）
其实onerror固然好但是还是有一类异常无法捕获。这就是网络异常的错误。
比如下面的例子。

<img src="./xxxxx.png">

试想一下我们如果页面上要显示的图片突然不显示了，而我们浑然不知那就是麻烦了。
addEventListener就是☟

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShO02JHlYV28UKYelkdicG6E77MIB3Ux3NviaK4Gic2UAY3VP9ZD79notvg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

运行结果如下☟

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShYCR6aKNyfoUsbQwKmEvicKXrjNO63LpAYK2ZaxouB7T1Vib2Al6kLGIw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**Promise异常捕获：**
Promise的出现主要是为了让我们解决回调地域问题。基本是我们程序开发的标配了。
虽然我们提倡使用es7 async/await语法来写。但是不排除很多祖传代码还是存在Promise写法。

new Promise((resolve, reject) => { abcxxx()});

这种情况无论是onerror还是监听错误事件都是无法捕获的。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShdfsRwoINe6dhna2lgDbuC9GqjiaQLqhLvUOJjSROrCrgkEjSfTpN4FA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

除非每个Promise都添加一个catch方法。
但显然，我们不能这样做。

window.addEventListener("unhandledrejection", e => { console.log('unhandledrejection',e)});

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSh5YvpGqRiaNeExseyjXMfib6zezwGl1kszX4SdC000M1am2cCJ5qzY7hA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们可以考虑将unhandledrejection事件捕获错误抛出交由错误事件统一处理就可以了。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShPGklLUSMbQzrz4pdyhoqaZCp1ibsocU2yiaMicWQCXE9NUPric5hIeELAg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**async/await异常捕获：**

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSho4Bka9ZBWn7p2sh8dNK8GSycwcPWNHWOWUSJeCG3XXkic4iakibvyEksA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

实际上async/await语法本质还是Promise语法。
区别就是async方法可以被上层的try/catch捕获。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShqibnK8k8rnTJxxXrtfxXRxmvKa2PywW9VZdg9HC6HIUoljWBkGGJ2IQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果不去捕获的话就会和Promise一样，需要用unhandledrejection事件捕获。
这样的话我们只需要在全局增加unhandlerejection就好了。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShdPABlm23StonItMQmN4IvdawcbF68IKg2BMrnyKCichCoJOtXb5uRiaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**小结：**

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShPLVgJI184tgRSq92UtvHuFGnDKe4icGjmhFZOQmqTC7jngSibZ6u95eg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

实际上我们可以将unhandledrejection事件抛出的异常再次抛出就可以统一通过error事件进行处理了。
最终用代码表示如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSh0waSSibvGKEf67n7wE9y14vicR7PeIAnPLmONZkmTeWIebBKsLM1TI5w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 三、前端工程化

**Webpack工程化：**

现在是前端工程化的时代,工程化导出的代码一般都是被压缩混淆后的。
比如：

setTimeout(() => {  xxx(1223)}, 1000)

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShfMURNF2sibgXjzbalMX5hboibtT7fViahvz2tTNBRf9NV3ZnicHT8SgQcA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

出错的代码指向被压缩后的JS文件，而JS文件长下图这个样子。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSh4zb6aOhFhQHqu5d09wDYaqWRHcJnyTm52tTfpfzPk5CVM5RNZ6H8DA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShzuT3zu6PuM68zKkqNcDRW3nLgLAukQ0J1szibOnNsYZdpQicFEbVhDRA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

如果想将错误和原有的代码关联起来，那就需要sourcemap文件的帮忙了。

**sourceMap是什么？**
简单说，sourceMap就是一个文件，里面储存着位置信息。
仔细点说，这个文件里保存的，是转换后代码的位置，和对应的转换前的位置。
那么如何利用sourceMap对还原异常代码发生的位置这个问题，我们到异常分析这个章节再讲。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShtCb2TdISViaibKvoLcauPtk6j1qCibSZ6xaTiaz5hzoZHWu4fDhd352hfA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

## 四、VUE、React创建工程

**VUE创建工程**

利用vue-cli工具直接创建一个项目。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShTa0CpDtlxvAOH4f0eBSH9mSTS3JJeXIQKqLicTe4WE43oDQ4CjaRbPQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

为了测试的需要我们暂时关闭eslint 这里面还是建议大家全程打开eslint。
在vue.config.js进行配置

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShISQHfry3DDgfjJm5lfyL8nTjr1RunCB2uibxLdlUtpmwFicO5DYLltwQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们故意在（文件位置☟）
src/components/HelloWorld.vue

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShk1Jw5rK75OBib0QsCEFyUukEic16RQ5icLBXws1nJSOrdvqmfeiaZ0LibLg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个时候 错误会在控制台中被打印出来,但是错误事件并没有监听到。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShTuz86uTiaKASrK85sCOl3fQO8JGI3B9noHVUibqsuR2KZBnmrYUr1g5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**handleError：**

为了对Vue发生的异常进行统一的上报，需要利用vue提供的handleError句柄。
一旦Vue发生异常都会调用这个方法。
我们在src/main.js

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShpFoJK5IM4R6eD1HZDj1wCfULkMBY8KfU0DbiaA9vJc1dRxDFaszhVEQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSh73vtrB4W9rh7uiaDT8552Llk9HhoN4RiaE9M0EscboKSqsE9bHSs5RQQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**React：**

npx create-react-app react-samplecd react-sampleyarn start

我们l用useEffect hooks 制造一个错误：

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSSh5HhGkfFr3CKDFqAQOc0qSozdVrz7fHg89KzePfbj9JHplaQzoj1cuQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

并且在src/index.js中增加错误事件监听逻辑：

window.addEventListener('error', args => {  console.log('error', error)})

但是从运行结果看虽然输出了错误日志但是还是服务捕获。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShwZnqzd3ae8cjcWItBZLcLAtSLGmL97hEAfPStRpcAEKib7jxDUiaiauQg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## 五、Error Boundary标签

**Error Boundary 标签**

错误边界仅可以捕获其子组件的错误。
错误边界无法捕获其自身的错误。
如果一个错误边界无法渲染错误信息，则错误会向上冒泡至最接近的错误边界。
这也类似于 JavaScript 中 catch {} 的工作机制。
创建ErrorBoundary组件

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShdIZAmFP4Oe8Tj7MYDqOD9jF8x8ichP0OmM6XGSLmQmDicpJyzFzwUCfg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在src/index.js中包裹App标签☟

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShSMfNZa4jDb1jykyQ1lpt0qB0iaRUT4b9K36lM1B1nKlGpmfREn6wv3w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最终运行的结果：

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zytY8ib4KHabjmVKLia2TSShMMKWLJ7ZjevNYw4hyicx6INX29jfbPUpKqdCMvg4lfyP4hQXCz6U75g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/CYlkzelz99Swhn5pknnFWF3BXbQyxkeaoZiaibDc30agxvu5KsOh4dHfRI9Mic4zibrXt7PLicB5OLPbOGgBKMWV8oQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## 六、异常上报

**动态创建img标签：**

其实上报就是要将捕获的异常信息发送到后端。最常用的方式首推动态创建标签方式。
因为这种方式无需加载任何通讯库，而且页面是无需刷新的。
基本上目前包括百度统计 Google统计都是基于这个原理做的埋点。

new Image().src ='http://localhost:7001/monitor/error'+ '?info=xxxxxx'

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2NI73dIX7INP0kNkicMULdKTicibTtIGyrUw80cobm30jsKpCvibduOv8BA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

通过动态创建一个img,浏览器就会向服务器发送get请求。
可以把你需要上报的错误数据放在querystring字符串中，利用这种方式就可以将错误上报到服务器了。

**Ajax上报：**
实际上我们也可以用ajax的方式上报错误，这和我们再业务程序中并没有什么区别。

**上报哪些数据：**

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2c37Zg3NGdTPwicvOyricIbWFZXXLCle9YN6ESrWkIv7KXiaqCArJUgCrA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

**上报哪些数据：**

我们先看一下error事件参数：

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2j9OJKlywzHmJU9MlyJfMs532T8KTm650MZsZa2SkQ7p16gOtreqFgg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其中核心的应该是错误栈，其实我们定位错误最主要的就是错误栈。
错误堆栈中包含了绝大多数调试有关的信息。其中包括了异常位置（行号，列号），异常信息

**上报数据序列化：**
由于通讯的时候只能以字符串方式传输，我们需要将对象进行序列化处理。
大概分成以下三步：1、将异常数据从属性中解构出来，存入一个JSON对象2、将JSON对象转换为字符串
3、将字符串转换为Base64

当然在后端也要做对应的反向操作 这个我们后面再说。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2Gk5JC9r1w1g1muJtEjvib8cIjE7TSaR1EUkKDct9ddNRhdeQC5650og/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## 七、异常上报后端服务器

**异常上报的后端服务器**

**搭建eggis工程：**

异常上报的数据一定是要有一个后端服务接收才可以。
我们就以比较流行的开源框架eggjs为例来演示

\# 全局安装egg-clinpm i egg-init -g # 创建后端项目egg-init backend --type=simplecd backendnpm i# 启动项目npm run dev

**编写error上传接口：**
首先在app/router.js添加一个新的路由

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME20O8yE36Y1EDagMmS0M2ZHYz1huRYibI3GeNiba2ia1bVlUicS9ibG7ic2ELw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

创建一个新的：controller (app/controller/monitor)

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME23ozJVmr3aVvs29uhAzAJk8qcwGUOL0hotGb5C8PN3fchJbBqxQYpDA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

看一下接收后的结果☟

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2M72DGl7XHmhibl6v44lI6efbiaVRKxW9Fw0oHcX562ric40f7yibpzBicFA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**记入日志文件：**

下一步就是讲错误记入日志。实现的方法可以自己用fs写，也可以借助log4js这样成熟的日志库。
当然在eggjs中是支持我们定制日志那么我么你就用这个功能定制一个前端错误日志好了。
在/config/config.default.js中增加一个定制日志配置

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME21Fvmz90L6ic1CElxIpibGnwrI04oHPibamEhMCzbKGxMvWNhyTnnzVegw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在/app/controller/monitor.js中添加日志记录：

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2hJlo0zwuwbxRGdSfxxSgzvQGbVgSyqTvHiaGLZR5cWdic3YLRKrHLI8Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**最后实现的效果：**

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME21iaHrxO59I81tAD5PyHgGCsaHLT7JmibNke0mm0BM055vJ4NBMePrnVg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)



## 八、Webpack插件实现SourceMap上报

**Webpack插件实现SourceMap上传**

谈到异常分析最重要的工作其实是将webpack混淆压缩的代码还原。

**创建Webpack插件：**

/source-map/plugin（文件位置）

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2XWL8ic771icVRzicoFWEIJ4Aqic1aoeWEx93a62NzkheQgfAJabpAGdngw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**加载webpack插件：**

webpack.config.js（文件位置）

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2YlPkLXSv7W1dDVkibBhUuxbSDs57ic1nzKTJdoKMZLm9gupKbpcNUy1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**添加读取sourcemap读取逻辑：**
在apply函数中增加读取sourcemap文件的逻辑/plugin/uploadSourceMapWebPlugin.js

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2RpCwXpNPW7O4Cf80HD0OXaBdqibVFgXsBPKLnOkbnV7WcERorZzdRwg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**实现http上传功能：**

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2bzjhLlAtLcia6sFRRtfukUQX0nYafaM9LLZfmicTNdh2wK4NYz5H3Bgg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**服务器端添加上传接口：**/backend/app/router.js（文件位置）

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2TnH5GexuakN5UMLwkh5lkd5tDD6lcdh6wuoiabsebDWWH2neTZZNFLA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**添加sourcemap上传接口：**/backend/app/controller/monitor.js

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2OGdYR7yVNc80a9X9JKialW8aKF6Nq0BxtNbpQYKL5RNmpPItmVDL43g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**最终效果：**
执行webpack打包时调用插件sourcemap被上传至服务器。

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2uNyylFTrCNdroaicPcbLlyQhia8uLiaaE5mSeFqMv39KjXvJNdArNyicJQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)



## 九、解析ErrorStack

**解析ErrorStack**

考虑到这个功能需要较多逻辑,我们准备把他开发成一个独立的函数并且用Jest来做单元测试：
先看一下我们的需求☟

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2IA4JadlMGWrggTrgYcib30ko0mJibmiaia1RtOJn0yqkWQlmGjOhXCU61w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**搭建Jest框架：**

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2FtiagsTEXviciagANXHCXnCnTJTRJM8aB0Ly6aEgFvaqM9zUibuEY40cwA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

首先创建一个/utils/stackparser.js文件☟

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2haXYOyTpy16ZCxUIlkaQHYwWKteJ3t4x07YGlvvVDP9bTzBdNm649w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在同级目录下创建测试文件stackparser.spec.js
以上需求我们用Jest表示就是

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2bGLibMWRUw9ZMHLhf84X7iaibuTcbm7Rp9dhWCedlLaMlSv6bZrZEBAng/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

整理如下:
下面我们运行Jest

npx jest stackparser --watch

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2AuCQyQAXsHSxsYZ93iaOPJxgia0uibfDl7PW3XiccxjGNbZjlgdCe9QwQQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

显示运行失败，原因很简单因为我们还没有实现对吧。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2ZAxenQIkibf034CCL5Ym7UlhSb5ePv4SuzkjgiaFf8mmTY3PexnT5ANQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

下面我们就实现一下这个方法☟

**反序列Error对象：**

首先创建一个新的Error对象 将错误栈设置到Error中。
然后利用error-stack-parser这个npm库来转化为stackFrame

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2n7bC7dA6ic4iajwgvnuC3farkwNic5TdvD7CmZrIJby9ewuXbD1SFOUIg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

运行效果如下☟

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2hhmMlgkpBPHJ2DVWMYpsluyqxgxfYLX1ylWzTwRvs92tibxSOkOicjVA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

**解析ErrorStack：**
下一步我们将错误栈中的代码位置转换为源码位置

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2WiaZFPKPGoPxkAZiagMDUEmUzdk6ZTvHD0aFR9WS6MLAxuVFgn1OzKHw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们再用Jest测试一下☟

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2ia3DUwTIRw9OFIlf80E0rxG9TBS9UcBnBNUcQqoT1wv5nOvkuiapE23g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这时我们再看一下结果：

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2GEGia6E9ribfw3cARnqkaBBk8DicICDWoZGBBKat1rL0rbxhiaibGdf4N0A/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

这样一来测试就通过啦~

**将源码位置记入日志：**

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME2GkWgVBa2Q4Duzp2qOEMj77OhthGuo47Ex0ZexIictUpt1cLaicciaTtLA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

记录完成后，我们再来看一下运行效果：

![img](https://mmbiz.qpic.cn/mmbiz_png/JCkibLOUr74zHT0AYnV6YGUcT5OudfME29Vo6BZVFQPDdlcibCkQvg2gPL4ArFSBE8p88u77sXh7xrrGAjMug61w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

结束了这一步，我们的ErrorStack工作就完成了。



## 十、两种开源框架

**需要运用的两种开源框架**

**Fundebug：**
Fundebug专注于JavaScript、微信小程序、微信小游戏、支付宝小程序、React Native、Node.js和Java线上应用实时BUG监控。 
自从2016年双十一正式上线，Fundebug累计处理了10亿+错误事件，付费客户有阳光保险、荔枝FM、掌门1对1、核桃编程、微脉等众多品牌企业。

**Sentry：**
Sentry 是一个开源的实时错误追踪系统，可以帮助开发者实时监控并修复异常问题。
它主要专注于持续集成、提高效率并且提升用户体验。
Sentry 分为服务端和客户端 SDK，前者可以直接使用它家提供的在线服务，也可以本地自行搭建；
后者提供了对多种主流语言和框架的支持，包括 React、Angular、Node、Django、RoR、PHP、Laravel、Android、.NET、JAVA 等。
同时它可提供了和其他流行服务集成的方案，例如 GitHub、GitLab、bitbuck、heroku、slack、Trello 等。
目前公司的项目也都在逐步应用上 Sentry 进行错误日志管理。

## 十一、总结

截止到目前为止，我们把前端异常监控的基本功能算是形成了一个MVP(最小化可行产品)。
后面需要升级的还有很多，对错误日志的分析和可视化方面可以使用ELK。
发布和部署可以采用Docker。对eggjs的上传和上报最好要增加权限控制功能。