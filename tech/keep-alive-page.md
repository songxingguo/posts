***

created: 2025-01-02 18:29
modified: 2025-01-02 18:29
Type: Extracts
tags:

* 编程/Vuex
  publish: true
  date: 2025-01-05
  Link: https://juejin.cn/post/7392105060900454411
  categories:
* 前端
  description: 当我们在列表页面进行了搜索操作，进入详情页面后再返回，我们会发现列表页面的搜索条件被清空了。第一时间我们想到的肯定是使用，Store存储搜索栏中的数据，当页面返回的时候再重新显示搜索数据，当然这没有问题，但相比于使用`keep-alive` 就显得有些多余了。在移动端，在进入详情页面之后，保留列表页面的状态是更为常见的需求。

> title: Vue 从列表页面进入详情页面，再次返回原页面，不清空查询条件或滚动位置
>
> Obsidian地址: obsidian://open?vault=content\&file=C%20Knowledge%2F%E5%89%8D%E7%AB%AF%2F%E8%81%8C%E4%B8%9A%E8%A7%84%E5%88%92%2F%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8%2F%E5%85%AB%E8%82%A1%E6%96%87%2FVue%20%E4%BB%8E%E5%88%97%E8%A1%A8%E9%A1%B5%E9%9D%A2%E8%BF%9B%E5%85%A5%E8%AF%A6%E6%83%85%E9%A1%B5%E9%9D%A2%EF%BC%8C%E5%86%8D%E6%AC%A1%E8%BF%94%E5%9B%9E%E5%8E%9F%E9%A1%B5%E9%9D%A2%EF%BC%8C%E4%B8%8D%E6%B8%85%E7%A9%BA%E6%9F%A5%E8%AF%A2%E6%9D%A1%E4%BB%B6%E6%88%96%E6%BB%9A%E5%8A%A8%E4%BD%8D%E7%BD%AE.md

path: tech/keep-alive-page

***

## 问题描述

当我们在列表页面进行了搜索操作，进入详情页面后再返回，我们会发现列表页面的搜索条件被清空了。第一时间我们想到的肯定是使用，Store存储搜索栏中的数据，当页面返回的时候再重新显示搜索数据，当然这没有问题，但相比于使用`keep-alive` 就显得有些多余了。在移动端，在进入详情页面之后，保留列表页面的状态是更为常见的需求。

`keep-alive`  解决问题的原理很简单，`keep-alive`是`vue`中的**内置组件**，能在组件切换过程中将状态保留在内存中，**防止重复渲染`DOM`**，关于`keep-alive`更多使用方法可以查看[keep-alive](https://cn.vuejs.org/guide/built-ins/keep-alive) 官方文档。

下面👇我们就使用 `keep-alive`  来更优雅的解决这个问题。

## 解决方法

我们的需求并不是每个页面都需要进行缓存，通常只有下面的情况我们需要缓存：

> 从 `首页` –> `列表页` –> `商详页` –> `返回到列表页(需要缓存)` –> `返回到首页(需要缓存)` –> `再次进入列表页(不需要缓存)`。

这时候可以按需来控制页面的 `keep-alive` ，我们可以在配置路由的时候添加 isKeepAlive 属性来实现。

```js
const router = new VueRouter({
    routes:[
        {
            path:'/',
            name:'home',
            component:home,
            meta:{isKeepAlive:true}
        },
        {
            path:'/news',
            name:'news',
            component:news,
            meta:{isKeepAlive:true}
        },
        {
            path:'/play',
            component:play,
            meta:{isKeepAlive:false}
        },
        ......
    ]
})
```

在渲染页面的时候，再根据`isKeepAlive` 的配置来使用 `<keep-alive>` 对页面进行缓存。

```html
<keep-alive v-if="route.meta.isKeepAlive">
 <router-view :key="route.path" />
</keep-alive>
<router-view :key="route.path" v-else />
```

如果你的每一个路由都是配置的不同页面组件，上面的代码已经你能完全满足需求了，不同的组件会根据 `isKeepAlive` 配置进行缓存。

但如果你遇到的问题也像我们的项目一样，每个页面都是通过动态生成的，也就是说所有的路由都指向同一组件 `@/views/index.vue` ，那就会出现所以页面都会被同时缓存或者同时不被缓存的情况，这显然不是我们配置想要达到对效果。

```js
const router = new VueRouter({
    routes:[
        {
            path:'/',
            name:'home',
            component: () => import('@/views/index.vue'),
            meta:{isKeepAlive:true}
        },
        {
            path:'/news',
            name:'news',
            component: () => import('@/views/index.vue'),
            meta:{isKeepAlive:true}
        },
        {
            path:'/play',
            component: () => import('@/views/index.vue'),
            meta:{isKeepAlive:false}
        },
        ......
    ]
})
```

为了解决这个问题可以在原组件外包装一层，使用路径作为唯一标识。通过 `isKeepAlive` 将需要缓存的页面添加到 `cacheRoutes` 中实现不同页面的缓存。

```js
import { h, ref } from 'vue'
import { type RouteLocationNormalizedLoadedGeneric } from 'vue-router'
// 用来存已经创建的组件
const storeComponents = new Map()
// 用来存需要缓存的路由
const cacheRoutes = ref<string[]>([])
// 原组件包里面
function formatComponent(component: object, route: RouteLocationNormalizedLoadedGeneric) {
  let afterComponent
  if (component) {
    const path = route.path
    if (storeComponents.has(path)) {
      afterComponent = storeComponents.get(path)
    } else {
      afterComponent = {
        name: path,
        render() {
          return h(component)
        },
      }
      if (route.meta.isKeepAlive) cacheRoutes.value.push(path) //进行缓存
      storeComponents.set(path, afterComponent)
    }
    return h(afterComponent)
  }
}
```

页面代码也需要改成了下面的样子，通过 `include` 指定需要缓存的组件名称。

```html
<router-view v-slot="{ Component, route }">
	<keep-alive :include="cacheRoutes">
	 <component :is="formatComponent(Component, route)" />
	</keep-alive>
</router-view>
```

## 结语

`keep-alive`  可以保证页面不刷新，不仅可以解决搜索框被清空的问题，还可以解决滚动了页面滚动条，进入其他页面后再返回，滚动条又回到了顶部的问题。

上面就是我对这个问题的最佳实践了，希望能对你有所帮助。
