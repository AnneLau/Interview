## 【译】JavaScript的工作原理：V8引擎内部+关于如何编写优化代码的5个技巧

妙堂传道者 前端瓶子君 *1周前*

> 来源：玄说前端 
>
> https://mp.weixin.qq.com/s/Jk9E_NvV6T4ryxQ2tuIWkw

几个星期前，我们开始了深入了解JavaScript及实际是如何运作的系列文章，我们认为通过了解JavaScript的构建模块以及它们如何共同发挥作用，您将能够编写更好的代码和应用程序。

本系列的第一篇文章

> https://juejin.im/post/6844903695298068488

重点介绍了引擎，运行时和调用堆栈的概述。第二篇文章将深入探讨谷歌V8 JavaScript引擎的内部部分。

## 概览

JavaScript引擎是一个程序或执行JavaScript代码的解释器。JavaScript引擎可以理解为标准解释器，或运行时编译器，它以某种形式将JavaScript编译为字节码。

- **V8——由Google开发的开源软件，用C ++编写**
- **Rhino——由Mozilla Foundation管理，开源，完全用Java开发**
- **SpiderMonkey ——第一个支持Netscape Navigator的JavaScript引擎，现在支持Firefox**
- **JavaScriptCore——开源，以Nitro销售，由Apple为Safari开发**
- **KJS - KDE的引擎，最初由Harri Porten为KDE项目的Konqueror Web浏览器开发**
- **Chakra (JScript9) ——Internet Explorer**
- **Chakra (JavaScript) ——Microsoft Edge**
- **Nashorn——由甲骨文Java语言和工具组开源作为OpenJDK的一部分**
- **JerryScript ——是物联网的轻量级引擎**

## V8为什么被创造出来？

V8引擎是由谷歌用C ++编写构建的开源程序。它在Google Chrome中被使用。但是，与其他引擎不同，V8也被用于流行的Node.js运行时。

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRjGxwibrCKrIdFJ6ZO1flSBJIWfh1sCPMwYlXzCmn6YRBicHCg8u1ZIfZa82jx6ic30YmnWgCrnN7dg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

V8最初设计旨在web浏览器内部执行JavaScript的性能提升，为了增加执行速度，V8没有把JavaScript代码转化成更有效的机器码，而不是使用解释器。像许多现代JavaScript引擎一样，如SpiderMonkey或Rhino（Mozilla），它通过实现JIT（即时）编译器将JavaScript代码编译成机器代码。这里的主要区别是V8不产生字节码或任何中间代码。

## V8曾经有两个编译器

在V8版本5.9出现之前（今年早些时候发布的），该引擎使用了两个编译器：

- **full-codegen**——一个简单而快速的编译器，可以生成简单但未被优化的机器代码。
- **Crankshaft**——一种更复杂的（即时）优化编译器，可生成高度优化的代码。

在V8引擎里面也使用了多个线程：

- 主线程：获取代码，编译代码然后执行它
- 还有一个被用来编译的单独的线程，因此主线程可以继续执行，而它也同时可以优化代码
- 一个分析线程，它将告诉运行时哪些方法耗费了大量的时间，以便**Crankshaft**可以优化它们
- 一些线程作用是处理扫描垃圾收集器

当JavaScript代码首次执行的时候，V8利用**full-codegen**直接将解析后的JavaScript转换为机器代码而无需其他中间过程的任何转换。这使它可以非常快速地开始执行机器代码。请注意，V8不使用中间字节码表示，因此无需解释器。

当你的代码运行了一段时间之后，这个分析线程已经收集了足够多的数据来告诉应该优化哪个方法。

接下来，Crankshaft优化从另一个线程开始，它把JavaScript抽象语法树转化为名为Hydrogen的高级静态单赋值（SSA）表示，并尝试优化Hydrogen图表，大多数优化都是在这个级别完成的。

## 内联

第一个优化是提前嵌入尽可能多的代码。嵌入就是被调用函数替换调用方法（调用函数的代码行）的过程。这个简单的步骤让后面的优化更有意义。

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRjGxwibrCKrIdFJ6ZO1flSBsH16aSWxbBias1K0TARNkzE9WOKpewGFicg5icIx4icdBr49yePiaeib6UCQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 隐藏类

JavaScript是一门基于原型的语言，它没有创建类，对象被创建是基于引用的，JavaScript也是一种动态编程语言，这意味着可以在实例化后轻松地在对象中添加或删除属性。

大多数的JavaScript解析器使用类似字典的结构（基于散列函数）来存储对象属性值在内存当中的位置，这个结构使得在JavaScript中检索属性的值比java或C#等非动态编程语言中的计算成本更高，在Java当中，所有对象属性都是在编译之前由固定对象模版确定的，并且无法在运行时动态添加或删除（C＃具有动态性类型，这是另一个主题），结果，属性值（或指向这些属性的指针）可以作为连续缓冲区存储在内存中，每个缓冲区之间具有固定偏移量，可以根据属性类型轻松确定偏移的长度。而在运行时可以更改属性值的JavaScript中，这是不可能的。

由于使用字典结构去查找属性值在内存当中的位置是非常低效的，V8使用来一个不同的方法去替代：**隐藏类**。隐藏类的作用类似于在Java语言中的固定对象模版（Classes），除非它们是在运行时创建的。让我们看看它们实际上是什么样的：

```
function Point(x, y) {
    this.x = x;
    this.y = y;
}
var p1 = new Point(1, 2);
```

一旦这个`new Point(1，2)`调用发生，V8将创建一个名为“C0”的隐藏类。![img](https://mmbiz.qpic.cn/mmbiz_jpg/pfCCZhlbMQRjGxwibrCKrIdFJ6ZO1flSBoH0sNH9JFwBJCBHLTqrDkmPk862xj7H5KEb5FiahFiamnjJS4xgjlcxQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

尚未为Point定义任何属性，因此“C0”为空。

一旦第一行代码`this.x = x`被执行（Point方法里面），V8将创建第二个基于“C0”的隐藏类“C1”，“C1”描述了可以找到属性x在内存中的位置（相对于对象指针），这种情况下，“x”被存储在偏移0处，这意味着当将内存中的Point对象视为连续缓冲区时，第一偏移位置将对应于属性“x”。V8还将使用“类转换”更新“C0”，类转换表示如果将属性“x”添加到Point对象，则隐藏类应从“C0”切换到“C1”。下面的Point对象的隐藏类现在是“C1”。

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRjGxwibrCKrIdFJ6ZO1flSBRFOhlGk2QPsaGfeuTF40NsuP8Jxlw6OIL4SsD1jy8WiamViapKgib6taA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> 每次将新属性添加到对象时，旧的隐藏类都会被更新到指向新隐藏类的转换路径。隐藏类转换非常重要，因为它们允许在以相同方式创建的对象之间共享隐藏类（比如实例化两个Point对象，他们的共同隐藏类是C0）。如果两个对象共享一个隐藏类并且同一属性被添加到它们中，则转换将确保两个对象都接收相同的新隐藏类（比如都添加“x”属性，就会都指向C1）以及所有的优化代码

当“this.y=y”被执行的时候，这个过程是重复进行的（Point函数里面的“this.y=y”），如果属性“y”被添加到Point上，类转换将会基于“C1”生成“C2”隐藏类，point对象的隐藏类将会更新到“C2”。

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRjGxwibrCKrIdFJ6ZO1flSBwcicBJYk2j7B36zE2AldFlRt6DznSXmvhHbG7dfrB2ddXqkyDXx8TuA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

隐藏类转换是决于属性添加到对象的顺序，看下下面的代码：

```
function Point(x, y) {
    this.x = x;
    this.y = y;
}
var p1 = new Point(1, 2);
p1.a = 5;
p1.b = 6;
var p2 = new Point(3, 4);
p2.b = 7;
p2.a = 8;
```

现在，假设对于p1和p2，将使用相同的隐藏类和转换。嗯，不是真的。对于“p1”，首先添加属性“a”，然后添加属性“b”。但是，对于“p2”，首先分配“b”，然后是“a”。因此，“p1”和“p2”以不同的隐藏类和不同的类转换结束。在这种情况下，以相同的顺序初始化动态属性要好得多（建议），以便可以重用隐藏的类。

## 内联缓存

V8优化动态类型语言的另一种方法称为**内联缓存**，内联缓存依赖于观察到对相同方法的重复调用往往发生在同一类型的对象上。可以在此处找到对内联缓存的深入解释。

我们将讨论一些内联缓存的概念（如果您没有时间查看上面的深入解释）。

### 那么它是怎样工作的？

V8维护了一个在最近的函数方法调用中作为参数传递的对象类型的缓存，并使用此信息来假设将来作为参数传递的对象类型。

如果V8能够对将传递给方法的对象类型做出很好的假设，它可以绕过确定如何访问对象属性的过程，而是使用先前查找到对象的所使用的隐藏类的存储信息。

### 那么隐藏类和内联缓存是如何相关的概念又是怎样的呢？

每当在特定对象上调用方法时，V8引擎必须执行对该对象的隐藏类的查找，以确定访问特定属性的偏移量。

在将同一方法成功调用两次到同一个隐藏类之后，V8会省略了隐藏类的查找，只是将属性的偏移量添加到对象指针本身。

对于该方法的所有将来的调用，V8引擎假定它的隐藏类未更改，并使用先前查找中存储的偏移直接跳转到特定属性的内存地址。这大大提高了执行速度。

内联缓存也是为什么相同类型的对象共享隐藏类非常重要的原因。

如果你创建两个相同类型和不同隐藏类的对象（正如我们之前的例子中所做的那样），V8将无法使用内联缓存，因为即使这两个对象属于同一类型，它们对应的隐藏类也会对其属性分配不同的偏移量。

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRjGxwibrCKrIdFJ6ZO1flSBgGmPK6ib3iaZVB9aKN3RU0BmoZ0icbcicEW5P4Xtqr4EzhF06bMT5D0Xyw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> 这两个对象基本相同，但“a”和“b”属性是按不同顺序创建的。

## 编译到机器代码

一旦Hydrogen图表优化完成，Crankshaft将其降低到被称为Lithium的低级别表示。

大多数Lithium实现都是依赖于整体架构的。寄存器分配发生在这一层上。

最后，Lithium被编译成机器代码。然后发生了一些叫做OSR的事情：堆栈替换(OSR)。

当我们开始编译和优化一个明显耗时的方法时，我们很可能之前一直在运行它。V8不会将它之前执行的很慢的代码抛在一边，再重新执行优化后的代码。相反，他会对这些慢代码所拥有的全部上下文（堆栈，寄存器）做一个转换，以便能够在执行这些慢代码的过程中直接切换到优化后的版本。

这是一项非常复杂的任务，请记住，在其他优化中，V8最初已经内联了代码。V8并不是唯一能够做到这一点的引擎。

有一种称为去优化的保护措施可以进行相反的转换，并在引擎作出的假设不再适用的情况下恢复到非优化代码。

## 垃圾回收

对于垃圾回收，V8是使用了传统的分代式标记清除垃圾回收机制来清除老一代，标记阶段JavaScript会停止执行，为了控制GC（垃圾回收）的成本和代码执行的稳定，V8是用来增量标记：和遍历整个堆、试图标记每一个可能的对象不同，它只是标记堆的一部分，然后恢复正常的执行，下一次GC将从上一次停止的地方继续遍历，在执行的时间段里，它允许短暂的暂停，如前文所说，这个清除阶段在单独的线程中进行的。

随着2017年早些时候V8 5.9版本的发布，一个新的执行管线被引入，这个新的管线在实际的JavaScript引用程序中实现了更大的性能提升和显著的内存节省。

这个新的管线是在V8的解释器Ignition和V8最新的优化编译器TurboFan之上构建的，

你可以在此查看V8团队有关该主题的博文。

> https://v8.dev/blog/launching-ignition-and-turbofan

自从V8的5.9版本发布之后，由于V8团队力争和新的JavaScript语言特性以及针对这些新特性所需要的优化保持一致，full-codegen和Crankshaft（这两项技术从2010年开始为V8服务）不再被V8用来运行JavaScript。

这意味着整个V8将拥有更简单和更易维护的架构。

![img](https://mmbiz.qpic.cn/mmbiz_png/pfCCZhlbMQRjGxwibrCKrIdFJ6ZO1flSByzSWP6XWeqW92vWhKTDd8kbyCmnoeyhXXYNJBncCPuibZUmL5iaon7mA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> Web和Node.js基准上的改进

这些优化只是刚刚开始，新的Ignition和TurboFan管线为未来的优化铺平了道路，未来JavaScript的性能会有更加巨大的提升，并能让V8在Chrome和Node.js中节约资源。

最后，这里提供一些小技巧，帮助大家写出更优化的、更优质的JavaScript。从上文中您一定可以轻松地总结出一些技巧，不过为了方便，仍然为您提供一份总结。

## 怎么写出最佳JavaScript代码

1.**对象属性的顺序：** 永远用相同的顺序为您的对象属性实例化，这样隐藏类和随后的优化代码才能共享。

2.**动态属性：** 在对象实例化后为其新增属性会导致隐藏类变化，从而会减慢为旧隐藏类所优化的方法的执行。所以，尽量在构造函数中分配对象的所有属性。

3.**方法：** 重复执行相同方法的代码会比不同的方法只执行一次的代码运行得更快（由于内联缓存的原因）。

4.**数组：** 避免使用keys不是递增数字的稀疏数组（sparse arrays）。并不为每个元素分配内存的稀疏数组实质上是一个hash表。这种数组中的元素比通常数组的元素会花销更大才能获取到。此外，避免使用预申请的大型数组。最好随着需要慢慢增加数组的大小。最后，不要删除数组中的元素，因这会使得keys变得稀疏。

5.**标记值：** V8用32个比特来表示对象和数字。它使用1个比特来区分是一个对象（flag = 1）还是一个整型（flag = 0）（被称为SMI或SMall Integer，小整型，因其只有31比特来表示值）。然后，如果一个数值大于31比特，V8就会给这个数字进行装箱操作（boxing），将其变成double型，并创建一个新的对象将这个double型数字放入其中。所以，为了避免代价很高的boxing操作，尽量使用31比特的有符号数。