# React 阻止事件冒泡失效、stopPropagation和stopImmediatePropagation分析，解决stopPropagation没有阻止冒泡问题



## 前言

做项目过程中， 发现了一个问题，onClik阻止冒泡事件并没有生效,仅仅只是阻止outClick，我的需求是只触发"inner dom click"，（PS:不想知道过程的可以直接跳过看最后的结论）

> **其他前端有趣的例子和坑合集：https://github.com/wqhui/blog**
> 代码链接：https://codepen.io/Lik_Lit/pen/OJLROMW

下面代码的输出是

> “root click”
> “inner dom click”
> “document click”

```js
const domContainer = document.querySelector('#root')

// 绑定在外层的点击 非documemt层的原生事件
domContainer.addEventListener('click', e => console.log('root click'))

class InnerDom extends React.Component {
  componentDidMount () {
    // documemt层的原生事件
    document.addEventListener('click', () => {
      console.log('document click')
    })
  }

  outClick = (e) => {
    console.log('out dom click')
  }

  innerClick = (e) => {
    e.stopPropagation();
    console.log('inner dom click')
  }

  render () {
    return <div onClick={this.outClick}>
      <button onClick={this.innerClick}> 测试冒泡</button>
    </div>
  }
}

ReactDOM.render(<InnerDom />, domContainer)
123456789101112131415161718192021222324252627282930
```

## 解决过程与坑

一寻思想起来React组件类似onClick的事件是合成事件，**本质上所有绑定是代理到document上的**，所有绑定都会冒泡到document层去执行。类似下面：

```js
// react所有合成事件, SyntheticEvent里面执行回调函数
document.addEventListener('click'， SyntheticEvent);

// 浏览器原生
document.addEventListener('click', () => {
   alert('document click');
})
1234567
```

所以在React绑定的合成事件调用`e.stopPropagation()`阻止的只是React里绑定的合成事件，比如我例子里面的*outClick*就没有触发。而对于原生事件还是阻止不了

百度告诉我说`e.nativeEvent.stopImmediatePropagation`可以阻止冒泡到原生事件，我就在在*innerClick*事件里加上了这句，输出变成了

> “root click”
> “inner dom click”

显然还是不可以，因为`stopImmediatePropagation`只是阻止同层级且绑定靠后的事件（具体参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/stopImmediatePropagation)），非document层的原生事件还是没有阻止，因为React的事件都是绑定在document层，**所以React要阻止冒泡事件只有使用原生的绑定去调用`e.stopPropagation()`**

```js
  refCb = (dom) => {
      //这里我就没有去解绑了
      dom && dom.addEventListener('click',this.innerClick)
  }
  <button ref={this.refCb} > 测试冒泡</button>
123456
```

这样就可以解决了

## 结论

1. React组件绑定事件是合成事件，**本质上是代理到document上**，采用事件冒泡的形式冒泡到document上面，然后React将事件封装给正式的函数处理运行和处理（可以理解成React所有事件都是绑定在document层）
2. 对于React的合成事件对象e，`e.stopPropagation()`只能阻止React合成事件的冒泡，`e.nativeEvent.stopImmediatePropagation`只能用来阻止冒泡到直接绑定在document上的事件
3. 要想阻止所有的冒泡事件，只能通过ref获得dom节点监听，用原生事件对象e的`e.stopPropagation()`去阻止冒泡