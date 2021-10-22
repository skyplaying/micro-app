我们分别列出基座应用和子应用需要进行的修改，具体介绍`micro-app`的使用方式。

### 基座应用

1、安装依赖
```bash
npm i @micro-zoe/micro-app --save
```

2、在入口处引入依赖
```js
// index.js
import microApp from '@micro-zoe/micro-app'

microApp.start()
```

3、分配一个路由给子应用
<!-- tabs:start -->

#### ** React **
```js
// router.js
import { BrowserRouter, Switch, Route } from 'react-router-dom'
import MyPage from './my-page'

export default function AppRoute () {
  return (
    <BrowserRouter>
      <Switch>
        // 👇👇 非严格匹配，/my-page/* 都将匹配到 MyPage 页面
        <Route path='/my-page'>
          <MyPage />
        </Route>
        ...
      </Switch>
    </BrowserRouter>
  )
}
```

#### ** Vue **

```js
// router.js
import Vue from 'vue'
import VueRouter from 'vue-router'
import MyPage from './my-page.vue'

Vue.use(VueRouter)

const routes = [
  {
    // 👇👇 非严格匹配，/my-page/* 都将匹配到 MyPage 页面
    path: '/my-page/*', // vue-router@4.x path的写法为：'/my-page/:page*'
    name: 'my-page',
    component: MyPage,
  },
]

export default routes
```
<!-- tabs:end -->

4、在`MyPage`页面中嵌入微前端应用
<!-- tabs:start -->

#### ** React **
```js
// my-page.js
export function MyPage () {
  return (
    <div>
      <h1>子应用</h1>
      // name(必传)：应用名称，每个`name`都对应一个应用，必须以字母开头，且不可以带有 `.`、`#` 等特殊符号
      // url(必传)：页面html的地址
      // baseroute(可选)：基座应用分配给子应用的基础路由，就是上面的my-page
      <micro-app name='app1' url='http://localhost:3000/' baseroute='/my-page'></micro-app>
    </div>
  )
}
```

#### ** Vue **
```html
<!-- my-page.vue -->
<template>
  <div>
    <h1>子应用</h1>
    <!-- 
      name(必传)：应用名称，每个`name`都对应一个应用，必须以字母开头，且不可以带有 `.`、`#` 等特殊符号
      url(必传)：页面html的地址
      baseroute(可选)：基座应用分配给子应用的基础路由，就是上面的my-page
     -->
    <micro-app name='app1' url='http://localhost:3000/' baseroute='/my-page'></micro-app>
  </div>
</template>
```
<!-- tabs:end -->

### 子应用

1、设置基础路由`(如果基座应用是history路由，子应用是hash路由，不需要设置基础路由，这一步可以省略)`

<!-- tabs:start -->

#### ** React **
```js
// router.js
import { BrowserRouter, Switch, Route } from 'react-router-dom'

export default function AppRoute () {
  return (
    // 👇👇 设置基础路由，子应用可以通过window.__MICRO_APP_BASE_ROUTE__获取基座下发的baseroute，如果没有设置baseroute属性，则此值默认为空字符串
    <BrowserRouter basename={window.__MICRO_APP_BASE_ROUTE__ || '/'}>
      <Switch>
        ...
      </Switch>
    </BrowserRouter>
  )
}
```

#### ** Vue **
```js
// mian.js
import Vue from 'vue'
import VueRouter from 'vue-router'
import routes from './router'

const router = new VueRouter({
  // 👇👇 设置基础路由，子应用可以通过window.__MICRO_APP_BASE_ROUTE__获取基座下发的baseroute，如果没有设置baseroute属性，则此值默认为空字符串
  base: window.__MICRO_APP_BASE_ROUTE__ || '/',
  routes,
})

let app = new Vue({
  router,
  render: h => h(App),
}).$mount('#app')
```
<!-- tabs:end -->


2、在webpack-dev-server的headers中设置跨域支持。
```js
devServer: {
  headers: {
    'Access-Control-Allow-Origin': '*',
  },
},
```

完成以上步骤微前端即可正常渲染。

上面列出了react和vue框架的使用方式，它们是可以自由组合的，比如基座应用是react，子应用是vue，或者基座应用是vue，子应用是react，或者基座应用和子应用都是react、vue。 `micro-app`对前端框架没有限制，任何框架都可以作为基座应用嵌入任何类型框架的子应用。


> [!NOTE]
> 1、url只是html地址，子应用的页面渲染还是基于浏览器地址的，关于这点请查看[路由一章](/zh-cn/route)
>
> 2、baseroute的作用请查看[路由配置](/zh-cn/route?id=路由配置)
>
> 3、子应用必须支持跨域访问，跨域配置参考[这里](/zh-cn/questions?id=_2、子应用静态资源一定要支持跨域吗？)
> 
> 4、子应用卸载时应该进行相应的卸载操作，参考[子应用卸载](/zh-cn/life-cycles?id=子应用卸载)
