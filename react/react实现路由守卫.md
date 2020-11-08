# react实现路由守卫

[![img](https://upload.jianshu.io/users/upload_avatars/19445607/571ab620-e1c2-4479-bea9-1542046a6188?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/b1d7f2bae488)

在开发中可能经常会遇到有关权限的问题,比如用户如果没有登陆的话是查看不了个人信息的.今天基于前面react-redux和react路由两篇文章的代码做一些修改来实现路由守卫的功能.
首先来修改一下src/store/login-redux.js

#### src/store/login-redux.js



```tsx
const initState = {
    login: false,
    loading: false
}
export const loginReducer = (state=initState, action) => {
    switch(action.type){
        case "nowlogin":
            return {
                login: false,
                loading: true
            }
        case "haslogin":
            return {
                login: true,
                loading: false
            }
        default :
            return state
    }
}

export const asyncLogin  = () => dispatch =>{
    dispatch({type: "nowlogin"});
    setTimeout(() => {dispatch({type: "haslogin"})}, 1500)
}
```

然后去修改src/routerdemo.js文件,在这只展示修改过的地方,其余代码和react路由文章中代码一样

#### src/routerdemo.js



```jsx
//页面引入登陆事件以及connent
import { connect } from "react-redux"
import { asyncLogin } from "./store/login-redux"
//首先我们来写权限控制组件,我们按照思路逐渐完善组件
//第一版我们需要写一个高阶组件让他返回一个Route组件,Route组件上面的属性我们需要根据redux中传过来的登陆状态动态判断应该展示的组件
function Permission ({component: Com,isLogin, ...rest})  {//因为组件名要大写所以需要给component取别名,isLogin登陆状态,rest组件其他属性
        return (
//Redirect中的to属性可以是字符串也可以是对象,我们在state属性中存储登陆后直接跳转的地址可在location中获取到
            <Route {...rest} render={({location, ...other}) => isLogin? <Com {...other}></Com> : <Redirect to={{pathname: "/login", state:{jumpUrl: location.pathname}}} ></Redirect>}></Route>
        )
    }
//第一版已经理清我们的思路了,那么缺少的就是登陆的状态了,我们接下来把组件完成
//因为Permission组件只需要登陆状态所以我们只需要给connect传入第一个参数就即可,connect也会返回一个组件我们只需要把他赋值给Permission
const Permission = connect(state => ({isLogin: state.login.login}))(
    ({component: Com,isLogin, ...rest}) => {
        return (
            <Route {...rest} render={({location, ...other}) => isLogin? <Com {...other}></Com> : <Redirect to={{pathname: "/login", state:{jumpUrl: location.pathname}}} ></Redirect>}></Route>
        )
    }
)
```

上面完成了对权限路由的处理,现在我们来处理登陆组件



```jsx
//第一版同样先理思路
function Login ({location, isLogin, loading, asyncLogin}){//location重定向到路由组件后要获取的地址
          //isLogin,loading都是登陆状态
          //asyncLogin异步登陆事件
        const fromUrl = location.state&&location.state.jumpUrl || "/"; //如果location.state中有jumpUrl就采用没有就跳转到首页
        if(isLogin){//如果登陆了就重定向到制定的页面,否则展示下面的登陆表单登陆
            return <Redirect to={fromUrl}></Redirect>
        }
        return(
            <div>
                <button onClick={asyncLogin}>{loading? "登录中..." : "登陆"}</button>
            </div>
        )
    }
//登陆组件的参数中我们还需要拿到isLogin, loading, asyncLogin,下来我们完善他
const Login = connect(state => ({
  isLogin: state.login.login, //登陆状态
  loading: state.login.loading}),//是否正在请求登陆
  {asyncLogin})(
    function ({location, isLogin, loading, asyncLogin}){
        const fromUrl = location.state&&location.state.jumpUrl || "/";
        if(isLogin){
            return <Redirect to={fromUrl}></Redirect>
        }
        return(
            <div>
                <button onClick={asyncLogin}>{loading? "登录中..." : "登陆"}</button>
            </div>
        )
    }
)
```

接下来就可以去使用我们的权限组件Permission了,我们需要对哪个组件做权限直接用Permission替换他,我们替换之前写的那个



```jsx
export default function Routerdemo(){
    return (
        <Router>
            <div style={{marginBottom:"15px"}}>
                <Link to="/">首页 </Link>&nbsp;&nbsp;&nbsp;&nbsp;
                <Link to="/about">关于</Link>&nbsp;&nbsp;&nbsp;&nbsp;
                <Link to="/person">用户中心</Link>&nbsp;&nbsp;&nbsp;&nbsp;
                <Link to="/search">发现</Link>
            </div>
            <Switch>
                <Route exact path="/" component={Home}></Route>
                <Route path="/about" component={About}></Route>
                <Permission path="/person" component={Person}></Permission>//将Person组件传进去
                {/* <Route path="/person" component={Person}></Route> */}
                <Route path="/login" component={Login}></Route>
                <Route component={Nopage}></Route>
            </Switch>
        </Router>
    )
}
```

查看页面效果



![img](https://upload-images.jianshu.io/upload_images/19445607-d21895b8afe82079.gif?imageMogr2/auto-orient/strip|imageView2/2/w/358/format/webp)

1.gif