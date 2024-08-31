---
created: 2024-08-30 11:12
modified: 2024-08-30 11:12
Type: Entity
tags:
  - 编程/Vue
publish: true
date: 2024-08-30 11:12
path: tech/addroutes
categories:
  - 前端
description: 在工作中遇到切换右侧的游戏，然后需要使用addRoutes动态添加对应的路由的情况。如果是添加不同路径的，没有什么问题，但当先后添加的路径一样，那么VueRouter 则不会使用后面添加的路径覆盖前面的路径，而是什么也不做。
title: addroutes 添加动态路由相同路径不覆盖问题
Obsidian地址: obsidian://open?vault=content&file=C%20Knowledge%2F%E5%89%8D%E7%AB%AF%2F%E5%BC%80%E5%8F%91%E6%8A%80%E6%9C%AF%2FVue%2FVue%E6%95%99%E7%A8%8B%2Faddroutes%20%E6%B7%BB%E5%8A%A0%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1%E7%9B%B8%E5%90%8C%E8%B7%AF%E5%BE%84%E4%B8%8D%E8%A6%86%E7%9B%96%E9%97%AE%E9%A2%98.md
---
## 目录
## 写在前面

![addroutes 添加动态路由相同路径不覆盖问题 20240831224941752](https://image.songxingguo.com/obsidian/20240831/addroutes%20%E6%B7%BB%E5%8A%A0%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1%E7%9B%B8%E5%90%8C%E8%B7%AF%E5%BE%84%E4%B8%8D%E8%A6%86%E7%9B%96%E9%97%AE%E9%A2%98-20240831224941752.webp)

在工作中遇到切换右侧的游戏，然后需要使用addRoutes动态添加对应的路由的情况。如果是添加不同路径的，没有什么问题，但当先后添加的路径一样，那么VueRouter 则不会使用后面添加的路径覆盖前面的路径，而是什么也不做。

```html
<router-link to="/">淘米客服平台</router-link>
```

也就导致在切换了游戏之后，当我们点击左侧的【淘宝客服平台】希望跳转到当前游戏的首页时，会跳转到上一个游戏的首页。主要就是因为当我们使用 addRoutes 添加为 `/`  添加新的 redirect 地址，并没有覆盖老的 redirect 地址导致的。

```js
router.addRoutes([{ path: '/', redirect: '/seer'}]);
```

下面👇是经过多方验证之后找到的最佳实践。

## 具体实现

因为公司项目是Vue2项目，因此 VueRouter 最高的版本只能使用 3.6.5，不能支持 [removeRoute](https://router.vuejs.org/zh/api/interfaces/Router.html#Methods-removeRoute) 先移除路由  `\`， 再重新添加路由 `\`。并且经过测试 [`addRoute` ](https://v3.router.vuejs.org/zh/api/#router-addroute)也并不能如文档所说【添加一条新的路由规则记录作为现有路由的子路由。如果该路由规则有 `name`，并且已经存在一个与之相同的名字，则会覆盖它】。最后我们只能另辟蹊径，通过重写 `addRoutes` 来实现我们需要覆盖相同路径的需求。

```js
const createRouter = (routes) => {
  return new VueRouter({
    mode: process.env.VUE_APP_ROUTER_MODE,
    base: process.env.BASE_URL,
    routes,
  });
};
 
const router = createRouter(routes);
 
// 重写addRoutes
const addRoute = VueRouter.prototype.addRoute;
VueRouter.prototype.addRoutes = function (newRoutes) {
  const routers = router.getRoutes();
  newRoutes = [...newRoutes, ...routers];
  router.matcher = createRouter(newRoutes).matcher; //通过重置matcher来重置router
  addRoute.call(this, ...newRoutes); //调用原有方法
};
```

上面代码中 `newRoutes = [...newRoutes, ...routers]` 就是最关键代码，主要是将新添加的路由放前面，在调用addRoute的时候，后面添加的相同路由就不会被`VueRouter`添加。又由于 `addRoutes` 官方已经==不推荐==使用了，因此在重写 addRoutes 时候我们可以使用 `addRoute.call(this, ...newRoutes)` 来实现路由的动态添加。 

## 写在后面

最后通过这次的问题，我学会当第三方插件不能满足我们的业务需求的时候，我们可以通过 `VueRouter.prototype.addRoute` 和 `addRoute.call(this, ...newRoutes)`重写插件的原有方法，从而达到满足我们业务需求的目的，相当于在原插件的基础上做了一下我们自己的封装。

当然需要注意的是直接在原型链上修改插件的原有方法不是什么时候都可取的，只有在完全确认，项目中所有使用 `addRoutes` 的地方都会使用重写后的逻辑才能直接修改插件方法，否则会对后续开发者的开发产生巨大的疑惑。

