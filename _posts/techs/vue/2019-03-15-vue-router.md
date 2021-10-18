---
title: Vue Router
date: 2019-03-15 15:20:35
permalink: /docs/techs/vue-router
key: vue-router
sharing: true
show_author_profile: true
show_subscribe: true
sidebar:
  nav: tech-nav
categories:
  - 前端
  - 技术文章
tags:
  - Vue.js
---

`Vue-Router`是`Vue.js`官方的路由管理器，与其深度集成。通过配置不同的路由，在`<router-view />`中渲染不同的组件。主要有以下几个知识点。
<!--more-->
# 动态路由匹配
通过动态路径参数，把某种模式匹配到的所有路由，全都映射到同个组件。而动态路径参数则是在定义路由是，在path上加上`:`冒号。例子：`{path:'/user/:id',component:User}`。其中`id`这个参数的值会被设置到`this.$route.params.id`中。
可以设置多段路径参数：

| 模式 | 匹配路径 | $route.params |
| --- | --- | --- |
| /user/:username | /user/evan | {username:'evan'} |
| /user/:username/post/:post_id | /user/evan/post/123 | {username:'evan',post_id:'123'} |

## 响应路由参数变化
当多次导航到相同路由，但是参数不同的路径时，组件实例会被复用，意味着组件生命周期钩子不会再被调用。想要对路由参数变化作出相应，有两种方法。
{:.info}

1. 通过`watch`监听`$route`对象。
1. 通过`beforeRouteUpdate`导航守卫。
```javascript
//watch
watch:{
  $route(to,from){
  }
}
//beforeRouteUpdate
beforeRouteUpdate(to,from,next){
  next()//使用此钩子一定要使用next()
}
```
## 通配符匹配
使用`*`能够匹配任意路径,并且`$route.params`内会自动添加一个参数`pathMatch`,表示被匹配的部分。
```javascript
// 给出一个路由 { path: '/user-*' }
this.$router.push('/user-admin')
this.$route.params.pathMatch // 'admin'
// 给出一个路由 { path: '*' }
this.$router.push('/non-existing')
this.$route.params.pathMatch // '/non-existing'
```
## 可选参数
通过在参数后面加`?`号，就能够使得这个参数是可选的。`{ path: '/optional-params/:foo?' }` `foo`参数是可选的，不传此参数也能够匹配到对应的组件。`{ path: '/optional-group/(foo/)?bar' }` `foo` 也是可选的。`/optional-group/bar` 和 `/optional-group/foo/bar`都能匹配到此组件。
## 正则参数
可以通过给参数添加正则来匹配组件。`{ path: '/params-with-regex/:id(\\d+)' }` `id`必须是数字才能匹配到此组件。
## 匹配优先级
同一个路径可能可以匹配多个路由，此时，哪个路由先定义，就先匹配谁。
# 嵌套路由
在实际业务场景中，视图层级可能是多层嵌套的。
```javascript
/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+
```
这时就可以使用嵌套路由，即在父组件中添加`children`数组，在数组里的组件会在父组件的`<route-view />`中渲染。
```javascript
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```


# 编程式导航与命名路由
命名路由意思是在`routes`配置中给某个路由设置名称。
```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```
编程式导航意思是不通过a标签来导航，而是使用`js`语句来导航。主要有以下方法。

1. `router.push(location,onComplete?,onAbort?)`

实际上`<router-link/>`内部实现也是使用这个方法。不过这是声明式的写法。   eg:

只有命名路由才能使用`params`参数,`query`在命名路由和指定路径时都可以用，会被解析为/name/?user=123
{:.warning}
```javascript
router.push('name');//字符串
router.push({path:'name'})//对象
router.push({name:'user',params:{userID:123}}) //命名路由
router.push({path:`/name/${id}`})//完整路径
router.push({path:'user',query:{userID:123}}) //解析为 /user/?userID=123
```
2.`router.replace(location,onComplete?,onAbort?)`
跟`push`的区别顾名思义。就是push是往`history`中添加，`replace`是直接替换，所以按返回并不能返回上一个页面。声明式路由也可以，只要在`<router-link :to=".." replace>`添加`replace`即内部使用`router.replace`导航。
3.`router.go(n)`
意思是在`history`记录中向前或向后几步。

`onComplete`:此回调函数将会在导航完成（在所有的异步钩子被解析之后）调用。
`onAbort`:此回调函数将会在导航终止 (导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由) 时调用。 如果目的地和当前路由相同，只有参数发生了改变 (比如从一个用户资料到另一个 `/users/1` -> `/users/2`)，你需要使用 [`beforeRouteUpdate`](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#%E5%93%8D%E5%BA%94%E8%B7%AF%E7%94%B1%E5%8F%82%E6%95%B0%E7%9A%84%E5%8F%98%E5%8C%96) 来响应这个变化 (比如抓取用户信息)
{:.success}


# 命名视图
同一个页面同时渲染不同的视图。可以使用命名视图。
你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 `router-view` 没有设置名字，那么默认为 `default`
```javascript
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```
配置`routes`时需要用`components`
```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {  //加上s
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```


## 嵌套命名视图
即嵌套路由与命名视图组合使用。
```javascript
{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions
  }, {
    path: 'profile',
    components: {
      default: UserProfile,
      helper: UserProfilePreview
    }
  }]
}
```


# 重定向和别名
## 重定向
通过`routes`配置，使用`redirect`来重定向路由。
```javascript
//1.直接redirect
routes:[
  {path:'/a',redirect:'/b'}
]

//2.命名redirect
routes:[
  {path:'/a',redirect:{name:'foo'}}
]

//3.带参数redirect
routes:[
  {path:'/a/:id',redirect:'/b/:id'}
 ]
//4.函数redirect
    { path: '/dynamic-redirect/:id?',
      redirect: to => {
        const { hash, params, query } = to
        if (query.to === 'foo') {
          return { path: '/foo', query: null }
        }
        if (hash === '#baz') {
          return { name: 'baz', hash: '' }
        }
        if (params.id) {
          return '/with-params/:id'
        } else {
          return '/bar'
        }
      }
 },

//5.区分大小写
{ path: '/foobar', component: Foobar, caseSensitive: true },

//6.redirect with pathToRegexpOptions
//sensitive: When true the route will be case sensitive. (default: false) true时区分大小写
//strict: When false the trailing slash is optional. (default: false) false时尾部斜杠在正则中为可有可无
//end: When false the path will match at the beginning. (default: true) false时正则将拥有，疑问：好像true才是拥有
//delimiter: Set the default delimiter for repeat parameters. (default: '/') 设置默认间隔符号
{ path: '/FooBar', component: FooBar, pathToRegexpOptions: { sensitive: true }},

// catch all redirect
{ path: '*', redirect: '/' }
```
## 别名`alias`
**`/a` 的别名是 `/b`，意味着，当用户访问 `/b` 时，URL 会保持为 `/b`，但是路由匹配则为 `/a`，就像用户访问 `/a` 一样。**
# 路由传参
在路由配置中使用`props` 可以将参数和路由解耦。参数会成为组件的`prop`
而`props`开启有三种模式。

1. 布尔模式：`props:true`;
1. 对象模式：即`props`有很多参数。
1. 函数模式：使用函数来返回`props`；
# 导航守卫
## 全局前置守卫`router.beforeEach`
```javascript
const router = new VueRouter({...})
router.beforeEach((to,from,next)=>{})
```
`to`：即将进入的目标对象
`from`：当前导航正要离开的路由
`next`：函数，一定要调用此方法来`resolve`这个钩子。参数可以是

1. `next()`：跳到下一个路由
1. `next(false)`：中断当前导航。
1. `next('/')、next({path:'/})、next({name:'index'})`：跳转到别的路由
1. `next(error)`：如果传入一个`error`实例，则导航会被终止且该错误会被传递给`router.onError()`注册过的回调。
## 全局解析守卫`router.beforeResolve`
这和 `router.beforeEach` 类似，区别是在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析之后**，解析守卫就被调用。
## 全局后置守卫`router.afterEach`
和前置守卫不同的是不接受`next()`,常用来设置一些`title`。
## 路由独享的守卫`beforeEnter`
只针对单个路由的守卫。
```javascript
const router = new VueRouter({
	routes:[
    {
      path:'login',
      component:login,
      beforeEnter:(to,from,next)=>{...}
    }
  ]
})
```
## 组件内的守卫

1. `beforeRouteEnter`
1. `beforeRouteUpdate`
1. `beforeRouteLeave`
```javascript
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
    //但是可以通过给next传递回调函数来访问组件实例,只有此守卫能够传递回调，下面两个不行，因为已经拥有实例了
    next(vm=>{})
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  }
}
```


## 完整的导航解析流程

1. 导航被触发。
1. 在失活的组件里调用 `beforeRouteLeave` 守卫。
1. 调用全局的 `beforeEach` 守卫。
1. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)。
1. 在路由配置里调用 `beforeEnter`。
1. 解析异步路由组件。
1. 在被激活的组件里调用 `beforeRouteEnter`。
1. 调用全局的 `beforeResolve` 守卫 (2.5+)。
1. 导航被确认。
1. 调用全局的 `afterEach` 钩子。
1. 触发 DOM 更新。
1. 用创建好的实例调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数。



# 路由原信息`meta`
`routes`配置中的每个路由对象叫做**路由记录** 。   通过`$route.marched`数组来获取路由记录中的`meta`字段。


# 过渡动效
使用`<transition></transition>`包裹`<router-view/>`添加类名，即可设置动态效果。使用方法参考vue过渡效果章节。
## 单个路由的过渡
在组件那使用`<transition></transition>`包裹组件，并设置不同的`name`
## 基于路由的动态过渡
```javascript
// 使用动态的 transition name 
<transition :name="transitionName">
  <router-view></router-view>
</transition>
watch: {
  '$route' (to, from) {
    const toDepth = to.path.split('/').length
    const fromDepth = from.path.split('/').length
    this.transitionName = toDepth < fromDepth ? 'slide-right' : 'slide-left'
  }
}
```
# 滚动行为

**注意: 这个功能只在支持 `history.pushState` 的浏览器中可用。**
{:.warning}
```javascript
const router = new VueRouter({
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return 
 这个方法返回滚动位置的对象信息，长这样：
    1.{ x: number, y: number }
    2.{ selector: string, offset? : { x: number, y: number }}
  }
})
第三个参数 savedPosition 当且仅当 popstate 导航 (通过浏览器的 前进/后退 按钮触发) 时才可用
```


## 滚动到锚点
```javascript
scrollBehavior (to, from, savedPosition) {
  if (to.hash) {
    return {
      selector: to.hash
    }
  }
}
```


## 异步滚动
```javascript
scrollBehavior (to, from, savedPosition) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({ x: 0, y: 0 })
    }, 500)
  })
}
```


# 路由懒加载

如果您使用的是 Babel，你将需要添加 [`syntax-dynamic-import`](https://babeljs.io/docs/plugins/syntax-dynamic-import/) 插件，才能使 Babel 可以正确地解析语法。
{:.warning}
```javascript
const foo = () => import ('./foo.vue')
```
## 把组件分块打包
有时候我们想把某个路由下的所有组件都打包在同个异步块 (chunk) 中。只需要使用 [命名 chunk](https://webpack.js.org/guides/code-splitting-require/#chunkname)，一个特殊的注释语法来提供 chunk name (需要 Webpack > 2.4)。
```javascript
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```
Webpack 会将任何一个异步模块与相同的块名称组合到相同的异步块中


# API
[VueRouter API](https://router.vuejs.org/zh/api/#router-link)
