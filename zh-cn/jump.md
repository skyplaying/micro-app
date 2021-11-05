
每个应用的路由实例都是不同的，应用的路由实例只能控制自身，无法影响其它应用，*包括基座应用无法通过控制自身路由影响到子应*。

常见的问题如：开发者想通过基座应用的侧边栏跳转，从而控制子应用的页面，这其实是做不到的，只有子应用的路由实例可以控制自身的页面。

要实现应用之间的跳转有两种方式：

## 方式一、history.pushState
通过[history.pushState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState)或[history.replaceState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/replaceState)进行跳转。

例如：
```js
history.pushState(null, null, 'page2')

// 主动触发一次popstate事件
window.dispatchEvent(new PopStateEvent('popstate', { state: null }))
```

对于hash路由也同样适用
```js
history.pushState(null, null, '#/page2')

// 主动触发一次popstate事件
window.dispatchEvent(new PopStateEvent('popstate', { state: null }))
```

> [!NOTE] 
> **注意事项:**
>
> 1、popstate事件是全局发送的，所有正在运行的应用都会接受到新的路由地址并进行匹配，此时要防止兜底到应用的404页面。
>
> 2、history.pushState并非适用于所有场景，一些框架如vue-router4，angular会出现问题，此时建议使用下面的方式2、3。


## 方式二、通过数据通信控制跳转
*适用场景: 基座控制子应用跳转。*

**子应用中监听数据变化**

```js
let oldPath = null
// 监听基座下发的数据变化
window.microApp.addDataListener((data) => {
  // 当基座下发跳转指令时进行跳转
  if (data.path && data.path !== oldPath) {
    oldPath = data.path
    router.push(data.path)
  }
})
```

**基座下发跳转指令**

```js
import microApp from '@micro-zoe/micro-app'

microApp.setData('子应用name', { path: '/new-path/' })
```

## 方式三、传递路由实例方法

*适用场景: 子应用控制基座跳转。*

**基座下发pushState函数：**
<!-- tabs:start -->

#### ** React **
```js
import { useEffect } from 'react'
import microApp from '@micro-zoe/micro-app'

export default (props) => {
  function pushState (path) {
    props.history.push(path)
  }

  useEffect(() => {
    // 👇 基座向子应用下发一个名为pushState的方法
    microApp.setData(子应用名称, { pushState })
  }, [])

  return (
    <div>
      <micro-app name='子应用名称' url='...'></micro-app>
    </div>
  )
}
```

#### ** Vue **

```html
<template>
  <micro-app
    name='子应用名称' 
    url='url'
    :data='microAppData'
  ></micro-app>
</template>

<script>
export default {
  data () {
    return {
      microAppData: {
        pushState: (path) => {
          this.$router.push(path)
        }
      }
    }
  },
}
</script>
```
<!-- tabs:end -->

**子应用使用pushState控制基座跳转：**

```js
window.microApp.getData().pushState(path)
```

