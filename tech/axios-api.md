---
created: 2024-09-13 21:15
modified: 2024-09-13 21:15
Type: Entity
tags:
  - 编程/网络/Axios
publish: true
date: 2024-09-17
path: tech/axios-api
categories:
  - 前端
description: Axios 是前端最常用的请求库，提供的功能也非常完善，能满足我们日常开发中的请求需要，但我们通常在项目里面都不会直接使用 Axios 库。我认为至少有两方面原因。首先，将项目中通用的请求逻辑进行封装，可以避免 DRY（不要重复自己） ，常见的通用逻辑包括请求拦截、响应拦截，异常的处理，请求取消、以及配置或数据格式的统一处理等。其次，对 Axios 库进行封装可以统一管理请求，提升了后续开发的可维护性。
title: 如何从零开始封装 Axios 请求
Obsidian地址: obsidian://open?vault=content&file=C%20Knowledge%2F%E5%89%8D%E7%AB%AF%2F%E8%81%8C%E4%B8%9A%E8%A7%84%E5%88%92%2F%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8%2F%E5%85%AB%E8%82%A1%E6%96%87%2F%E5%A6%82%E4%BD%95%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%B0%81%E8%A3%85%20Axios%20%E8%AF%B7%E6%B1%82.md
---
## 目录
## 写在前面

Axios 是前端最常用的请求库，提供的功能也非常完善，能满足我们日常开发中的请求需要，但我们通常在项目里面都不会直接使用 Axios 库。

我认为至少有两方面原因。首先，将项目中通用的请求逻辑进行封装，可以避免 DRY（不要重复自己） ，常见的通用逻辑包括请求拦截、响应拦截，异常的处理，请求取消、以及配置或数据格式的统一处理等。其次，对 Axios 库进行封装可以统一管理请求，提升了后续开发的可维护性。

每个项目具体的业务场景以及后端接口不一致，会导致对`api.js`的封装有细微的差别，但通过我多年的开发经验来看，将API封装成类的形式是最佳实践。

下面让我们按照请求流程，从零开始封装属于自己的Axios请求。

![](https://ask.qcloudimg.com/http-save/yehe-2451713/493e8a52e82a82f90da86cff3629771d.png)

## 封装Api类

首先，我们从创建一个最简单的 Api 类开始，分别定义 request、get、post、put 等请求方法，其中 `_createApi`  为私有方法用于根据配置创建 `Axios` 请求实例，并添加请求拦截器和响应拦截器。

```js
import axios from 'axios'
import defaultConfig from './config.js' // 默认配置

class Api {
  static _createApi (config) {
    const instance = axios.create({
      ...defaultConfig,
    })
    Api.setRequestInterceptor(instance);
    Api.setResponseInterceptor(instance);
    return instance(config);
  }

  static request(config =  {}) {
    return Api._createApi(config);
  }

  static get (url, config = {}) {
    return Api._createApi({
      url,
      method: 'GET',
      ...config
    })
  }

  static post (url, data = {}, config = {}) {
    return Api._createApi({
      url,
      data,
      method: 'POST',
      ...config
    })
  }

  static put (url, data = {}, config = {}) {
    return Api._createApi({
      url,
      data,
      method: 'PUT',
      ...config
    })
  }
}

export default Api
```

下面是 `config.js` 文件中的默认配置。

```js
export default {
  method: 'post',
  // 基础url前缀
  baseURL: process.env.VUE_APP_BASE_API,
  // 请求头信息
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  // 参数
  data: {},
  // 设置超时时间
  timeout: 10000,
  // 携带凭证
  withCredentials: true,
  // 返回数据类型
  responseType: 'json'
}
```

## 请求参数处理

有的时候，我们传递的参数并不是完全符合 `Axios` 的配置格式，那我们就需要在 `Axios` 创建请求之前将参数处理成统一格式或者是添加一下额外的配置，比如 `skipErrorHandler`。

```js
static _createApi (config) {
	config = transformRequestConfig(config);
	//...
	return instance(config);
}
```

具体实现就是，在我们在创建 `_createApi` 的时候， 使用 `transformRequestConfig` 方法将参数处理成  `Axios` 的配置格式。

```js
function transformRequestConfig (params) {
  const config = {
    url: params.request.url || '',
    method: params.request.method,
    skipErrorHandler: params.skipErrorHandler,
    request: params.request,
  }
  // 配置处理
  //...
  return config;
}
```

## 请求拦截器

请求拦截器中可以做很多事情，比如修改请求头，请求取消等。修改请求头比较简单，下面主要讲讲怎么取消请求。

```js
// request 拦截器
static setRequestInterceptor(instance) {
  instance.interceptors.request.use(
	config => {
	
	  config = Api.addCancelToken(config);

	  return config
	},
	error => {
	}
  )
}
```

### 取消请求

在切换页面后，取消之前还未完成的axios请求，可以避免之前的请求结果影响当前页面的判断并减少不必要的请求消耗。

实现思路就是将当前页面的所以请求取消方法存在state中，然后在router.beforeEach()钩子函数中遍历执行所有的取消方法。

首先，我们在 store 中定义一个 `cancelTokenArr` 用于存储当前页面的所有的取消请求方法。接着定义 `ADD_CANCEL_TOKEN`  和 `CLEAR_CANCEL_TOKEN` 分别用于向数组 `cancelTokenArr`  中添加取消方法 和 通过遍历数组 `cancelTokenArr`  来取消当前页面的所有请求。

```js
const state = {
  cancelTokenArr: []
}

const mutations = {
	  ADD_CANCEL_TOKEN(state, cancel) {
      if (!state.cancelTokenArr) {
        state.cancelTokenArr = []
      }
      if (cancel) {
        state.cancelTokenArr.push(cancel)
      }
    },
    // 取消所有请求
    CLEAR_CANCEL_TOKEN(state) {
      state.cancelTokenArr.forEach(c => {
        if (c) {
          c()
        }
      })
      state.cancelTokenArr = []
    }
}

export default {
  namespaced: true,
  state,
  mutations,
}
```

然后，我们就可以在请求拦截器中添加 `addCancelToken` ，用于在每次发起请求的时候，将当前请求的取消方法添加到数组中。

```js
/**
* 为请求配置对象添加取消令牌
* @param {Object} config - 请求配置对象
* @see store.commit
* @see axios.CancelToken
*/
static addCancelToken(config){
	config.cancelToken = new axios.CancelToken((cancel) => {
	  app.$store.commit('http/ADD_CANCEL_TOKEN', cancel)
	})
	return config;
}
```

最后，我们就可以在 `router.beforeEach` 钩子中，监听页面的切换，从而取消当前页面的所有请求。一定要注意在最后执行`next()`方法，否则路由不会继续执行。

```js
router.beforeEach((to, from, next) => {
  // 切换路由时先取消所有请求
  store.commit('http/CLEAR_CANCEL_TOKEN')
  next()
})
```

需要注意的是，取消请求会出现 error 错误，如果我们不希望报错，则可以通过 `axios.isCancel` 来判断当前请求是否是被取消的，来进行一些特殊的处理。

```js
err => {
	if (axios.isCancel(err)) {
	  console.warn('Request canceled');
	  return Promise.reject({isCancel: true, ...err})
	} 
	// ...
	return Promise.reject(err) // 返回接口返回的错误信息
}
```

## 响应拦截器

响应拦截器中，通常是进行错误处理和数据处理。

```js
// response 拦截器
static setResponseInterceptor(instance) {
	instance.interceptors.response.use(
	  response => {
		const { skipErrorHandler = false } = response.config;
	
		const data = response.data;
	
		if(data.result == 0 || skipErrorHandler) return transformResponseData(data, response.config);
	
		// 根据返回的code值来做不同的处理,根据接口来配置错误码
		const errMsg  = data.err_desc && data.err_desc.message || data.err_desc;
		const errCode = data.err_desc && data.err_desc.errorCode;
		
		Api.handleGeneralError(data.result, errMsg);
		Api.handleBusinessError(errCode, errMsg);
	
		return data
	  },
	  err => {
		//...
	
		// 跳到404
		app.$router.push('/404')
		return Promise.reject(err) // 返回接口返回的错误信息
	  }
	)
}
```

### 错误处理

常见的错误，包括网络错误、系统错误和业务错误。通常的做法是将这三种错误全部放在响应拦截器中进行处理，并且使用 `switch-case`  的语法来处理不同的状态码。

```js
switch (errStatus) {
	case 400:
		errMessage = '错误的请求'
		break
	case 401:
		errMessage = '未授权，请重新登录'
		break
	case 403:
		errMessage = '拒绝访问'
		break
	// ...
	default:
		errMessage = `其他连接错误 --${errStatus}`
}
```

如果错误处理不多，这样完全没有问题，但一旦错误状态过多，就会导致响应拦截器中显得臃肿，后续的可维护性就变得极其差。

为了解决这个问题，首先，我们可以按照错误类型进行分类处理，将不同类型错误处理拆分成不同的函数，分别处理不同的错误类型。其次，每种错误类型处理我们尽可能用==程序逻辑==去替代人脑逻辑，通常的做法就是使用Map数据结构，这是一种非常常用的做法，需要逻辑判断的地方都能使用Map数据结构进行处理。

```js
/**
* 处理业务错误
* @param {String} errCode - 错误码
* @param {String} errMsg - 错误信息
*/
static  handleBusinessError = (errCode, errMsg) => {
	const bErrMap = {
	  '10000': {
		msg: '审核中',
		dangerouslyUseHTMLString: true,
	  },
	}
	
	const defaultError = {  
	  msg: errMsg,
	  dangerouslyUseHTMLString: true,
	} 
	
	const error = bErrMap[errCode] || defaultError;
	
	const options = {
	  title: error.title || '提示信息',
	  confirmButtonText: '确定',
	  dangerouslyUseHTMLString: error.dangerouslyUseHTMLString,
	}
	
	const {title, ...restOptions} = options;
	error.msg && app.$alert(error.msg, title, restOptions);
	
	return bErrMap.hasOwnProperty(errCode);
}
```

```js
/**
* 处理通用错误
* @param {String} errCode - 错误码
* @param {String} errMsg - 错误信息
*/
static  handleGeneralError = (errCode, errMsg) => {
	const gErrMap = {
	  '-1': {
		msg: '系统错误，请联系网站开发组'
	  },
	  '-2': {
		handler: () => {
		}
	  },
	  '-3': {
		msg: '您还没有该操作权限',
	  },
	  '-4': {
		msg: '参数错误：' + errMsg,
		dangerouslyUseHTMLString: true,
	  },
	  '-5': {
		hasBusinessError: true
	  },
	  '-6': {
		title:'系统错误',
		msg: '服务配置错误'
	  },
	}
	
	const defaultError = {  
	  msg: '未知错误',
	} 
	
	const error = gErrMap[errCode] || defaultError;
	
	const options = {
	  title: error.title || '提示信息',
	  confirmButtonText: '确定',
	  dangerouslyUseHTMLString: error.dangerouslyUseHTMLString,
	}
	const {title, ...restOptions} = options;
	if(error.hasBusinessError) return false
	error.msg && app.$alert(error.msg, title, restOptions);
	error.handler && error.handler();
	return true
}
```

上面的 `handleBusinessError` 和 `handleGeneralError` 处理方法都大同小异，通过 `errCode` 获取到当前错误，然后就可以进行错误处理或者提示。

如果部分请求需要忽略拦截器的全局错误处理，通常可以在config中传递`skipErrorHandler` 的方式进行处理。

```js
// 响应拦截器
request.interceptors.response.use(
  (response: AxiosResponse) => {
    const { skipErrorHandler } = response.config;
    //...
    return res;
  },
  (error: AxiosError) => {
    const { skipErrorHandler } = error.config;
    //...
    return Promise.reject(error);
  },
);
```

### 数据处理

接口返回的数据有时候并不是我们真正想要的数据结构或者存在数据层级嵌套会比较深的情况。这样就可以通过 `transformResponseData` 将数据处理成标准的结构之后，才返回给业务层使用，这样会极大的减少业务中大量的冗余代码。

```js
function transformResponseData({ data }, config) {
  const request = config.request;
  const paths = request.data.key.map(item => {
    //...   
    return {
      dataPath,
      paramsPath
    };
  });
  const dataObj = getDataByPath(data, paths[0].dataPath);
  const paramsObj = getDataByPath(data, paths[0].paramsPath);
  return  {
    data: getArray(dataObj),
    total: parseInt(dataObj.total),
    pageSize: parseInt(paramsObj.page_size),
    resData: data
  }
}
```

如果要获取接口数据中比较深层次的数据，这里可以使用 `Lodash` 的 `get`，通过传递对象属性路径获取到对应的属性。

```js
import { get, isPlainObject } from 'lodash-es';
/**
 * 通过路径获取数据
 * 这个函数使用 `lodash` 的 `get` 方法来安全地从一个嵌套对象中获取指定路径的数据。
 * 如果对象中存在该路径，则返回对应的值；如果不存在，则返回一个空数组。
 * @param {Object} res - 要搜索的源对象
 * @param {String} path - 表示对象属性路径的字符串，使用点表示法
 * @return {*} - 在对象中找到的属性值或空数组（如果属性不存在）
 */
export function getDataByPath(res, path, defaultValue = []) {
  return get(res, path, defaultValue);
}

/**
 * 从对象中获取数组属性
 * @param {Object} obj - 要检查的对象
 * @returns {Array} - 对象中的第一个数组属性，如果没有找到则为一个空数组
 */
export function getArray(obj)  {
  if(!isPlainObject(obj)) return obj;
  const key = Object.keys(obj).find(key => Array.isArray(obj[key]));
  return obj[key] || [];
}
```

## 如何使用

我们将Api 挂载到`Vue.prototype`上，这样就可以很方便的在全局进行访问。

```js
import Api from './api'

const install = Vue => {
  if (install.installed) return
  install.installed = true

  Vue.prototype.$Api = Api
}

export default install
```

经过上面一系列的封装，在业务中使用请求就非常简单了，只需要将 ` await this.$Api.request(params)` 放入 `try-catch`
块中，正常的代码逻辑按顺序执行，错误逻辑则可以在`catch`块中进行处理。

```js
async loadData(reload = false) {
  try {
    //...
    let params = this.searchParams;
    this.$store.commit('base/CHANGE_LOADING', true);
    const { data, total, pageSize, resData } = await this.$Api.request(params);
    //...
  } catch (error) {
    console.error(error);
  } finally { 
    this.$store.commit('base/CHANGE_LOADING', false);
  }
},
```

## 完整代码

下面是`api.js` 的完整代码，这里只是为大家提供一些自己在项目中封装 `Axios` 的一些思想方法，每个项目的具体需求可能不太一样，很多地方不能做到开箱即用，而是需要根据自己的项目进行调整。

```js
import axios from 'axios' // 先安装
import defaultConfig from './config.js' // 倒入默认配置
import qs from 'qs' // 序列化请求数据，视服务端的要求
import {
  app
} from '.././main'
import { getDataByPath, getArray } from '@/utils/data';

function transformRequestConfig (params) {
  const config = {
    url: params.request.url || '',
    method: params.request.method,
    skipErrorHandler: params.skipErrorHandler,
    request: params.request,
  }
  //...
  return config;
}

function transformResponseData({ data }, config) {
  const request = config.request;
  const paths = request.data.key.map(item => {
    //...
    return {
      dataPath,
      paramsPath
    };
  });
  const dataObj = getDataByPath(data, paths[0].dataPath);
  const paramsObj = getDataByPath(data, paths[0].paramsPath);
  return  {
    data: getArray(dataObj),
    total: parseInt(dataObj.total),
    pageSize: parseInt(paramsObj.page_size),
    resData: data
  }
}

class Api {
  /**
   * 为请求配置对象添加取消令牌
   * @param {Object} config - 请求配置对象
   * @see store.commit
   * @see axios.CancelToken
   */
  static addCancelToken(config){
    config.cancelToken = new axios.CancelToken((cancel) => {
      app.$store.commit('http/ADD_CANCEL_TOKEN', cancel)
    })
  }

  // request 拦截器
  static setRequestInterceptor(instance) {
      instance.interceptors.request.use(
        config => {

          Api.addCancelToken(config);

          return config
        },
        error => {
          //...
          return Promise.reject(error) 
        }
      )
  }

   /**
   * 处理业务错误
   * @param {String} errCode - 错误码
   * @param {String} errMsg - 错误信息
   */
   static  handleBusinessError = (errCode, errMsg) => {
    const bErrMap = {
      '10000': {
        msg: '审核中',
        dangerouslyUseHTMLString: true,
      },
    }

    const defaultError = {  
      msg: errMsg,
      dangerouslyUseHTMLString: true,
    } 

    const error = bErrMap[errCode] || defaultError;

    const options = {
      title: error.title || '提示信息',
      confirmButtonText: '确定',
      dangerouslyUseHTMLString: error.dangerouslyUseHTMLString,
    }

    const {title, ...restOptions} = options;
    error.msg && app.$alert(error.msg, title, restOptions);

    return bErrMap.hasOwnProperty(errCode);
  }

  /**
   * 处理通用错误
   * @param {String} errCode - 错误码
   * @param {String} errMsg - 错误信息
   */
  static  handleGeneralError = (errCode, errMsg) => {
    const gErrMap = {
      '-1': {
        msg: '系统错误，请联系网站开发组'
      },
      '-2': {
        handler: () => {

        }
      },
      '-3': {
        msg: '您还没有该操作权限',
      },
      '-4': {
        msg: '参数错误：' + errMsg,
        dangerouslyUseHTMLString: true,
      },
      '-5': {
        hasBusinessError: true
      },
      '-6': {
        title:'系统错误',
        msg: '服务配置错误'
      },
    }

    const defaultError = {  
      msg: '未知错误',
    } 

    const error = gErrMap[errCode] || defaultError;

    const options = {
      title: error.title || '提示信息',
      confirmButtonText: '确定',
      dangerouslyUseHTMLString: error.dangerouslyUseHTMLString,
    }
    const {title, ...restOptions} = options;
    if(error.hasBusinessError) return false
    error.msg && app.$alert(error.msg, title, restOptions);
    error.handler && error.handler();
    return true
  }

  // response 拦截器
  static setResponseInterceptor(instance) {
     instance.interceptors.response.use(
      response => {
        const { skipErrorHandler = false } = response.config;

        const data = response.data;

        if(data.result == 0 || skipErrorHandler) return transformResponseData(data, response.config);

        // 根据返回的code值来做不同的处理,根据接口来配置错误码
        const errMsg  = data.err_desc && data.err_desc.message || data.err_desc;
        const errCode = data.err_desc && data.err_desc.errorCode;
        
        Api.handleGeneralError(data.result, errMsg);
        Api.handleBusinessError(errCode, errMsg);
        
        return data
      },
      err => {
        if (axios.isCancel(err)) {
          console.warn('Request canceled');
          return Promise.reject({isCancel: true, ...err})
        } 

        // 跳到404
        app.$router.push('/404')
        return Promise.reject(err) // 返回接口返回的错误信息
      }
    )
  }

  static _createApi (config) {
    // 提交携带cookie
    axios.defaults.withCredentials = true
    const instance = axios.create({
      ...defaultConfig,
    })
    Api.setRequestInterceptor(instance);
    Api.setResponseInterceptor(instance);
    return instance(config);
  }

  static request(config =  {}) {
    config = transformRequestConfig(config);
    return Api._createApi(config);
  }

  static get (url, config = {}) {
    return Api._createApi({
      url,
      method: 'GET',
      ...config
    })
  }

  static post (url, data = {}, config = {}) {
    return Api._createApi({
      url,
      data,
      method: 'POST',
      ...config
    })
  }

  static put (url, data = {}, config = {}) {
    return Api._createApi({
      url,
      data,
      method: 'PUT',
      ...config
    })
  }
}

export default Api
```

这次封装中最为重要的一点就是将请求封装成Api类了和使用了单一职责的思想方法，也就是在封装的过程中尽量将每个功能点都拆分成方法。我想在这种思想方法的框架下，其余的都是小问题了。

当然，除了上面提到的一些常见的封装，大家也可以在此基础上根据业务需求进行不断迭代。