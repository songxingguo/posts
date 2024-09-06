---
created: 2024-09-03 18:14
modified: 2024-09-03 18:14
Type: Entity
tags:
  - 编程/Vue
publish: true
date: 2024-09-03 18:14
Coding地址: https://songxingguo.coding.net/p/taomee/d/kf.taomee.com_client/git/tree/dev-songxingguo-auth/src/utils
path: tech/vauth
categories:
  - 前端
description: 权限校验是后台管理系统中必不可少的功能，常用的做法有两种，第一种，在数据返回后，对数据进行整体清洗， 通过遍历每个按钮的权限并保存到disabled变量中，然后再在页面中将 disabled 应用到按钮组件上。第二种，就是在每次渲染按钮组件的时候再进行权限校验。
title: 如何实现一个自定义权限校验指令v-auth
Obsidian地址: obsidian://open?vault=content&file=C%20Knowledge%2F%E5%89%8D%E7%AB%AF%2F%E8%81%8C%E4%B8%9A%E8%A7%84%E5%88%92%2F%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8%2F%E5%85%AB%E8%82%A1%E6%96%87%2F%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E8%87%AA%E5%AE%9A%E4%B9%89%E6%9D%83%E9%99%90%E6%A0%A1%E9%AA%8C%E6%8C%87%E4%BB%A4v-auth.md
---
## 目录
## 写在前面

权限校验是后台管理系统中必不可少的功能，常用的做法有两种，第一种，在数据返回后，对数据进行整体清洗， 通过遍历每个按钮的权限并保存到disabled变量中，然后再在页面中将 disabled 应用到按钮组件上。第二种，就是在每次渲染按钮组件的时候再进行权限校验。

![v auth 20240906223658496](https://image.songxingguo.com/obsidian/20240906/v-auth-20240906223658496.webp)

第一种看上去好像也不错，但至少有两个问题需要注意，第一，如果我们是对列表数据的操作按钮进行权限校验，那我们就不得不对所有列表数据进行遍历，也就有了下面一般`ugly`的代码。

```js

for (var i = 0; i < rightmainData.tableData.length; i++) {
if (rightmainData.tableData[i].handleButton.length != 0) {
  for (var j = 0; j < rightmainData.tableData[i].handleButton.length; j++) {
	var permission rightmainData.tableData[i].handleButton[j].permiss;
	if (permission && !app.$store.state.user.authority.hasOwnProperty(permission)) {
	  rightmainData.tableData[i].handleButton[j].disabled = true;
	  rightmainData.tableData[i].handleButton[j].havePermission = false
	} else {
	  rightmainData.tableData[i].handleButton[j].havePermission = true
	}
  }
}
}
```

不仅如此，还有一个更大的问题是，如果我们的权限校验是依赖其他数据，每次数据改变的时候，我们都需要写新的方法来重新计算权限。

既然这样，那我们为什么不在需要进行鉴权的地方再进行权限校验并且每次数据变更后再重新计算呢？在Vue中最适合干这个事情的非自定义指令莫属，自定义指令也至少有两个优点，可以将它绑定到任何需要添加额外功能的DOM上，并且自定义指令拥有 update、install等事件钩子，可以在虚拟DOM更新的时候触发更新，这和权限校验需要动态更新的需求完美契合。

下面👇我要做的就是填前人留下的坑，将上面`ugly`的代码用v-auth来进行彻底重构，只需要在需要鉴权的元素上，添加一个v-auth指令即可，并且当数据改变时，还能重新进行权限校验。

## 定义指令

首先，我们定义一个`v-auth` 指令 ,  这个指令拥有一个validate方法，当被绑定元素插入父节点或虚拟DOM和绑定的数据更新时，就会被调用并且重新校验权限。

```js
import Vue from 'vue';
import store from '@/store'

 const VAuth = {
  install(Vue) {
    Vue.directive('auth', {
      inserted: (el, bind) => {
        this.validate(el, bind);
      },
      update: (el, bind) => {
        this.validate(el, bind);
      },
    });
  },
  validate(el, bind) {
    const { disabled, forbidden, havePermission } = this.calcPermission(bind);
    const  isHidden = this.hiddenEL(el, bind, forbidden)
    this.setDisabled(el, disabled);
    this.setDataset(el, {
      forbidden,
      havePermission,
      disabled,
      isHidden,
    })
  },
};

Vue.use(VAuth);
```

上面的 `validate` 方法一共做了四件事情，首先， calcPermission 计算了用户权限，根据得到的权限，hiddenEL 用于隐藏元素，setDisabled 用于禁用按钮，最后再使用 setDataset 将权限信息绑定到 dataset 上。

### 权限校验

权限校验部分和业务强相关，每一个项目具体实现可能不太一样，但基本思想都是类似的，就从接口获取到当前用户所有的权限信息 authority，然后再判断一下当前元素的权限是否在用户的权限信息里面。像我们的项目里面，会比较复杂一点，还有后端配置的 `disabled` 和 需要根据接口数据动态计算的禁用条件 `forbidden` 。

```js
calcPermission(bind) {
	const { value: options = {} } = bind;
	const permissionColumn = 'permiss';
	const permission = options[permissionColumn];
	// 权限为空或者操作权限包含在用户权限中
	const havePermission = !permission || store.state.user.authority.hasOwnProperty(permission);
	const forbiddenFuncName =  {
	  from: 'dialogFormBtnDisabled',
	  table: 'btnDisabled',
	}[options.type] || 'btnDisabled';
	const forbidden = options.forbidden && options.data && Vue.prototype.$utiloptions.forbidden, options.data, options.tableType || false;
	const disabled = !havePermission || options.disabled || forbidden;
	return {
	  disabled,
	  forbidden,
	  havePermission,
	}
},
```
### 隐藏元素

当用户没有权限的时候，通常有两种做法，一种是将元素直接隐藏掉，另一种则只是禁用掉元素。首先讲讲隐藏元素，因为我们默认是对元素进行禁用，需要通过修饰符中的 `hidden` 值或者 `arg` 来判断是不是需要开启对元素的隐藏。紧接着，就可以使用上一步计算得到的 `forbidden` 和 `display:none` 来隐藏元素。

这里需要注意的一点，当我们需要重新显示元素的时候，需要通过之前元素的状态还原回去，像我们项目中，会存在`inline-block` 的元素，因此我们需要将class设置了`inline`的元素还原到 `display:inline-block`。

```js
// 据修根据参数或者饰符hidden控制显示隐藏
hiddenEL(el, bind, forbidden) {
	const { modifiers = {}, arg } = bind;
	const hidden = arg === true || modifiers.hidden;
	const isHidden = hidden && forbidden;
	const isInline = el.classList.contains('inline');
	const display = isInline ? 'inline-block' : '';
	el.style.display = isHidden ? 'none' : display;
	return isHidden;
},
```

### 禁用元素

禁用元素就比较简单了，首先，为元素设置`disabled` 属性，让元素本身无法响应点击事件，然后为元素添加禁用样式，我们这里使用的是ElementUI，因此只需要为元素添加上 `is-disabled` 的类名即可。

```js
setDisabled(el, disabled) {
	if(disabled) {
	  el.setAttribute('disabled', disabled);
	  el.classList.add("is-disabled");
	} else  {
	  el.removeAttribute('disabled');
	  el.classList.remove("is-disabled");
	}
},
```

### 共享数据

官方文档里面明确指出，除了el 之外，我们不能修改其他属性，如果需要共享数据就只能通过`dataset`。


>  
> 除了 `el` 之外，其它参数都应该是**只读的**，切勿进行修改。如果需要在**钩子之间共享数据**，建议通过元素的 vue 自定义组件 dataset|dataset 来进行

具体做法如下，首先，在 `v-auth` 指令中，将需要传递到外部的数据绑定到 el.dataset 上。

```js
setDataset(el, datasetMap) {
	Object.keys(datasetMap).forEach(key => {
	  el.dataset[key] = !!datasetMap[key];
	})
}
```

然后，我们就可以在组件中通过ref获取到元素上的 `dataset` 属性。这里需要注意的是，我们获取到的true和false的字符串的，需要转换成布尔值，这里可以 使用`JSON.parse` 实现。

```js
mounted() {
  const disabled = this.$refs.formItemRef.dataset.disabled;
  this.disabled = JSON.parse(disabled);
  console.log(this.disabled) // true
}
```

### 校验节流

到这里功能都基本上实现了，但我们在 update 钩子里面添加权限校验 `validate` 时，不只有数据变化会触发钩子，绑定元素的虚拟DOM改变都会触发，这显然不是我们想要的。这里可以使用 `lodash`  的 isEqual 来判断绑定的数据有没有改变，只当数据发生改变时，我们才重新进行权限校验，从而达到节流的目的。

```js
import _ from 'lodash-es';

update: (el, bind) => {
   if(this.isEqual(bind)) return;
   this.validate(el, bind);
},

isEqual(bind) {
	const { value, oldValue } = bind;
	return _.isEqual(value, oldValue);
},
```

## 使用指令

下面就是指令使用的三种方法，第一种没有任何修饰符的 `v-auth`， 这种情况下，当用户没有权限时，元素会被禁用掉。

```js
<el-button v-auth="AuthOptionsSchema" >{{ item.labelText }}</el-button>
```

第二种用法可以为指令添加一个 `hidden` 修饰符，当用户没有权限时，元素会被隐藏掉。

```html
<div :class="['kf-form-item', inline ? 'inline' : '']" ref="formItemRef" v-auth.hidden="AuthOptionsSchema">
</div>
```

第三种和第二种类似，只是这里的 `hidden` 是动态传递的，而不是写死的。

```html
<el-button v-auth:[item.hidden]="AuthOptionsSchema" >{{ item.labelText }}</el-button>
```

需要特别注意的一点是，我们给指令绑定的是一个对象，对象的地址是不变的，当改变对象内的值并不会触发 `update` 方法。我们可以通过 `JSON.parse(JSON.stringify(data))` 对绑定的对象进行深拷贝，来触发更新。

```js
computed: {
	AuthOptionsSchema() {
	  const data =  {
		...this.formItem,
		type: 'from',
		data: this.formList
	  }
	  return JSON.parse(JSON.stringify(data))
	}
},
```

