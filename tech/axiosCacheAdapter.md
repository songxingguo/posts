---
created: 2024-10-17 17:02
modified: 2024-10-17 17:02
Type: Entity
tags:
  - 编程/网络/Axios
  - 文章/前端最佳实践
publish: true
date: 2025-01-19
Link: https://www.cnblogs.com/longmo666/p/17996836
Coding地址: https://songxingguo.coding.net/p/taomee/d/kf.taomee.com_client/git/tree/master/src/http/axiosCacheAdapter
path: tech/axiosCacheAdapter
categories:
  - 前端
description: 这篇文章其实是 [05. 如何从零开始封装 Axios 请求]的续集，在上一篇文章中讲了如何从零封装一个 Axios 请求，这次是对请求功能的补充，增加缓存功能，避免相同的请求多次请求服务器。
title: 07.  Axios 如何数据缓存防止重复请求
Obsidian地址: obsidian://open?vault=content&file=C%20Knowledge%2F%E5%89%8D%E7%AB%AF%2F%E8%81%8C%E4%B8%9A%E8%A7%84%E5%88%92%2F%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8%2F%E5%85%AB%E8%82%A1%E6%96%87%2F07.%20%20Axios%20%E5%A6%82%E4%BD%95%E6%95%B0%E6%8D%AE%E7%BC%93%E5%AD%98%E9%98%B2%E6%AD%A2%E9%87%8D%E5%A4%8D%E8%AF%B7%E6%B1%82.md
---

## 目录

> 点击链接查看[原文](https://blog.songxingguo.com/posts/tech/axiosCacheAdapter "原文")，订阅[SSR](https://blog.songxingguo.com/atom.xml "SSR")。

## 写在前面

这篇文章其实是 [05. 如何从零开始封装 Axios 请求](https://blog.songxingguo.com/posts/tech/axios-api) 的续集，在上一篇文章中讲了如何从零封装一个 Axios 请求，这次是对请求功能的补充，增加缓存功能，避免==相同的请求==多次请求服务器。

在我们的项目中遇到了同一个页面中有多个表格，每个表格都需要重复请求同一个接口的请求而且传参都一样的问题。为了避免在极短时间内重复请求，则使用浏览器缓存上次的值，再次请求时不再请求接口，直接返回缓存的值，并且上次的值在指定时间内自动清除。

## 具体实现

值得一提的是适配器模式可以在不改变已有的接口的情况下，通过包装一层的方式实现功能的扩展，比如这里就是通过[适配器](https://github.com/axios/axios/blob/v1.x/lib/adapters/README.md)将原本需要从后端接口获取数据更改为从缓存中获取数据， 下面是 axiosCacheAdapter.js文件的具体实现：

```js
import axios from 'axios';

const cache = new Map();
const EXPIRATION_TIME_MS = 1 * 1000; // 缓存过期时间（例如：1 秒）

// 轮询检查是否有数据
function checking(callback, validate) {
  setTimeout(() => {
      const res = validate && validate()
      if(res) {
        callback && callback(res)
      } else  {
        checking(callback, validate)
      }
  }, 200);
}

function CacheValue(response = null) {
  return {
    timestamp: Date.now(),
    response,
  }
}

const cacheAdapterEnhancer = config => {
  const { url, method, params, data } = config;
  const cacheKey = JSON.stringify({ url, method, params, data });

  if (cache.has(cacheKey)) {
    let cachedResult = cache.get(cacheKey);
    if (Date.now() - cachedResult.timestamp < EXPIRATION_TIME_MS) {
      return new Promise((resolve) => {
        const validate = () => {
          const cachedResult = cache.get(cacheKey);
          return cachedResult.response;
        }
        const callback = (res) => {
          resolve(res);
        }
        checking(callback, validate)
      });
    } else {
      cache.delete(cacheKey);
    }
  }

  cache.set(cacheKey, CacheValue());

  return axios.defaults.adapter(config).then((response) => {
    cache.set(cacheKey, CacheValue(response));
    return response;
  });
};

export default cacheAdapterEnhancer;
```

上面的代码通过轮询的方式获取前一次请求的结果，直到前一次请求的结果返回之后再停止执行。

而且还使用 `Map` 数据结构，通过它键唯一的特性，达到存储不同请求的目的。

> \[!WARNING] 注意❗
> 注意上面代码的 `axios` 版本为 `0.19.0` ,   其他更新版本可能会报错 ` axios.defaults.adapter not function` ，比如`1.7.9`有多种默认adapter，需要通过 `axios.getAdapter(axios.defaults.adapter[0])(config)` 获取默认adapter：
>
> ```js
> return axios.getAdapter(axios.defaults.adapter[0])(config).then((response) => {
> 	cache.set(cacheKey, CacheValue(response));
> 	return response;
> });
> ```

然后在主应用中引入并使用这个适配器：

```js
import axios from 'axios';
import cacheAdapterEnhancer from './axiosCacheAdapter';

// 创建axios实例并配置缓存适配器
const instance = axios.create({
  adapter: cacheAdapterEnhancer,
});

// 使用自定义的axios实例发送请求
instance.get('/your-api-endpoint', { params: yourParams })
.then(response => {
 console.log(response.data);
})
.catch(error => {
 console.error(error);
});
```
