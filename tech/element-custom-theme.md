---
created: 2025-01-23 14:35
modified: 2025-01-23 14:35
Type: Entity
tags:
  - 编程/UI库/ElementUI
publish: true
date: 2025-01-23 14:35
官方文档: https://element.eleme.cn/#/zh-CN/component/custom-theme#strong-xiu-gai-bian-liang-strong
Demo 地址: https://element.eleme.cn/#/zh-CN/theme
categories:
  - 前端
path: tech/element-custom-theme
description: 动态切换不同的主题色是项目中非常常见的功能，这次我们公司的后台管理系统中也需要实现这个功能。
title: 08. Vue2+ElementUI项目怎么实现动态切换主题色
Obsidian地址: obsidian://open?vault=content&file=C%20Knowledge%2F%E5%89%8D%E7%AB%AF%2F%E5%BC%80%E5%8F%91%E6%8A%80%E6%9C%AF%2FElementUI%2F08.%20Vue2%2BElementUI%E9%A1%B9%E7%9B%AE%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0%E5%8A%A8%E6%80%81%E5%88%87%E6%8D%A2%E4%B8%BB%E9%A2%98%E8%89%B2.md
---

## 目录

> 点击链接查看[原文](https://blog.songxingguo.com/posts/tech/element-custom-theme "原文")，订阅[SSR](https://blog.songxingguo.com/atom.xml "SSR")。

## 写在前面

动态切换不同的主题色是项目中非常常见的功能，这次我们公司的后台管理系统中也需要实现这个功能。我们项目使用的是Vue2+ElementUI+scss的技术栈，切换主题的常见方法有三种：

- 可以通过工具生成不同主题css文件，切换主题的时候动态切换css主题文件。如果不需要动态切换主题，也可以通过环境变量打包不同的主题，[theme-chalk-preview](https://elementui.github.io/theme-chalk-preview) 就是这样实现的。
- 当然，也可以全部主题样式都打包在一起，然后通过classname来切换。
- 最后，还可以通过CSS3变量实现ElementUI主题的切换。

最后一种，是我认为最好的一种方案，虽然可能有一些兼容性问题，但如果不需要兼容IE，这是一个完美的解决方案，即使要兼容IE，我觉得问题也不大，不过就是降级体验，使用默认的主题颜色而已。

这里重点讨论第三种方案，然后再说说怎么通过环境变量更改不同的主题。

## 方案一：使用 CSS3 变量实现 ElementUI 主题的动态切换

### 效果演示

最终的效果可以在[预览地址](https://dlillard0.github.io/element-custom-theme/)中查看，也可以在[element-custom-theme](https://github.com/DLillard0/element-custom-theme) 中查看源码。

### 实现思路

我们都知道 ElementUI 的主题是可以通过scss变量来动态改变的，如果我们紧紧是为了在编译的时候修改 ElementUI 组件的主题样式，只需要定义一个 `element-variables.scss` 文件修改一下主题变量就可以了，但如果想要在运行时动态修改项目的主题色，就需要在此基础上通过 CSS3 变量来实现，这样是因为原本的scss变量，只支持编译时指定，但并不支持后期改变，也就不能完全满足我们动态切换主题的需求。

实现方式很简单，就三步：

1. 使用 css 变量定制 ElementUI sass 变量

```css
// element-variables.scss
--color-primary: #409eff;
$--color-primary: var(--color-primary, #409EFF);
```

2. 覆盖 sass 内置的 mix 函数，使用 css 的 color-mix 函数代替。这里之所以要使用 color-mix 函数代替内置的 mix 函数，主要是因为内置的 mix 函数不支持 `var(--color-primary, #409EFF)` 作为参数，会导致编译通不过。

```css
// element-variables.scss
@function mix($color1, $color2, $p: 50%) {
  @return color-mix(in srgb, $color1 $p, $color2);
}
```

3. 在页面注入 css 变量，通过 js 动态==修改 css 变量==即可实现整个 ElementUI 的主题切换。

```js
const root = document.documentElement;
root.style.setProperty('--color-primary', color);
```

### 具体实现

`element-variables.scss` 文件的完整代码如下，下面的代码首先在 `:root` 中定义css变量，然后将ElemententUI 的主题变量使用 css 变量进行定义，覆盖一下内置函数，最后就是修改ElementUI 主题的常规操作，定义字体文件路径和引入主题样式，想要了解更多修改ElementUI主题的内容可以查看[官方文档](https://element.eleme.cn/#/zh-CN/component/custom-theme)。

```css
:root {
  --color-white: #fff;
  --color-primary: #409eff;
  --color-primary-light-1: color-mix(in srgb, var(--color-white) 10%, var(--color-primary));
  --color-primary-light-2: color-mix(in srgb, var(--color-white) 20%, var(--color-primary));
  --color-primary-light-3: color-mix(in srgb, var(--color-white) 30%, var(--color-primary));
  --color-primary-light-4: color-mix(in srgb, var(--color-white) 40%, var(--color-primary));
  --color-primary-light-5: color-mix(in srgb, var(--color-white) 50%, var(--color-primary));
  --color-primary-light-6: color-mix(in srgb, var(--color-white) 60%, var(--color-primary));
  --color-primary-light-7: color-mix(in srgb, var(--color-white) 70%, var(--color-primary));
  --color-primary-light-8: color-mix(in srgb, var(--color-white) 80%, var(--color-primary));
  --color-primary-light-9: color-mix(in srgb, var(--color-white) 90%, var(--color-primary));
  --color-success: #67c23a;
  --color-warning: #e6a23c;
  --color-danger: #f56c6c;
  --color-info: #909399;
}
// 覆盖 sass 内置的 mix 函数，使得其支持var变量
@function mix($color1, $color2, $p: 50%) {
  @return color-mix(in srgb, $color1 $p, $color2);
}
/* 改变主题色变量 */
$--color-white: var(--color-white) !default;
$--color-primary: var(--color-primary) !default;
$--color-primary-light-1: var(--color-primary-light-1) !default;
$--color-primary-light-2: var(--color-primary-light-2) !default;
$--color-primary-light-3: var(--color-primary-light-3) !default;
$--color-primary-light-4: var(--color-primary-light-4) !default;
$--color-primary-light-5: var(--color-primary-light-5) !default;
$--color-primary-light-6: var(--color-primary-light-6) !default;
$--color-primary-light-7: var(--color-primary-light-7) !default;
$--color-primary-light-8: var(--color-primary-light-8) !default;
$--color-primary-light-9: var(--color-primary-light-9) !default;
$--color-success: var(--color-success) !default;
$--color-warning: var(--color-warning) !default;
$--color-danger: var(--color-danger) !default;
$--color-info: var(--color-info) !default;

/* 改变 icon 字体路径变量，必需 */
$--font-path: "~element-ui/lib/theme-chalk/fonts";
@import "~element-ui/packages/theme-chalk/src/index";
```

如果需要在组件或其他样式文件中使用ElementUI的主题变量，直接引入`element-variables.scss`之后，直接使用即可。

```css
<style lang="scss">
@import "@/styles/element-variables.scss";

.el-button {
  background: $--color-primary;
}
</style>
```

最后，这里我们使用 `el-color-picker` 组件来选择主题颜色，然后通过修改主题变量来实现主题的动态切换。这里还使用了`process.env.VUE_APP_PRIMARY_COLOR`定义了一个默认主题色，通过 `localstorage` 存储切换后的主题色，下次进入项目的时候首先获取 `localstorage` 中的主题色。

```js
<template>
<el-color-picker v-model="primaryColor" :predefine="predefineColors" show-alpha size="mini" @change="updateThemeColor"></el-color-picker>
</template>
<script>
export default {
	data() {
		return {
            primaryColor: getStorage('primaryColor') || process.env.VUE_APP_PRIMARY_COLOR,
          predefineColors: [
              '#ff4500',
              '#ff8c00',
              '#ffd700',
              '#90ee90',
              '#00ced1',
              '#1e90ff',
              '#c71585',
              'rgba(255, 69, 0, 0.68)',
              'rgb(255, 120, 0)',
              'hsv(51, 100, 98)',
              'hsva(120, 40, 94, 0.5)',
              'hsl(181, 100%, 37%)',
              'hsla(209, 100%, 56%, 0.73)',
              '#c7158577'
            ]
		};
	},
	  beforeMount() {
	    this.updateThemeColor(this.primaryColor)
	  },
	methods: {
	    updateThemeColor(color) {
	      if(!color) {
	        color = process.env.VUE_APP_PRIMARY_COLOR
	        this.primaryColor = color
	      }
	      const root = document.documentElement;
	      root.style.setProperty('--color-primary', color);
	      setStorage('primaryColor', color)
	    }
	}
};
</script>
```

### 兼容性处理

如果你的项目不需要考虑IE浏览器的兼容性，或者可以允许IE降级体验，不需要动态切换主题，上面的内容已经能完全满足你需求了。但如果你非要IE 也需要切换主题，由于 css 变量和 color-mix 函数存在一些兼容性问题，可以通过写一个 postcss 插件来提供兜底方案。这样在不支持 css 变量和 color-mix 函数的浏览器中，就会使用固定的色值。

下面是 `postcss-plugin.js` 文件的完整定义：

```js
//postcss-plugin.js
const Color = require('color')

// 处理 CSS3 变量和 color-mix 兼容性问题
// 将
// .el-alert--success.is-light {
//   background-color: color-mix(in srgb, #FFFFFF 90%, var(--color-success, #67C23A));
//   color: var(--color-success, #67C23A);
// }
//
// 转换为
// .el-alert--success.is-light {
//   background-color: #F0F9EB;
//   background-color: color-mix(in srgb, #FFFFFF 90%, var(--color-success, #67C23A));
//   color: #67C23A;
//   color: var(--color-success, #67C23A);
// }
// 这样在不支持 css 变量和 color-mix 函数的浏览器中，就会使用固定的色值
module.exports = (opts = {}) => {
  return {
    postcssPlugin: 'POSTCSS-PLUGIN',
    Declaration (decl, { Declaration }) {
      let newVal = decl.value

      const varArr = getVar(decl.value)
      if (varArr) {
        varArr.forEach(i => {
          const _i = i.match(/,(.*)\)/)
          if (_i) newVal = newVal.replace(i, _i[1].trim())
        })
      }

      const mixArr = getColorMix(newVal)
      if (mixArr) {
        mixArr.forEach(i => {
          const _i = getColorMixDefault(i)
          if (_i) newVal = newVal.replace(i, _i)
        })
      }

      if (newVal !== decl.value) {
        decl.before(new Declaration({ prop: decl.prop, value: newVal }))
      }
    }
  }
}

function getVar (value) {
  return value.match(/var\([^(]+(\([^(]+\))?[^(]*\)/g)
}

function getColorMix (value) {
  return value.match(/color-mix\([^(]+(\([^(]+\))?([^(]+\([^(]+\))?[^(]*\)/g)
}

function getColorMixDefault (value) {
  const arr = value.match(/[^,]+,([^(]+|[^(]+\([^)]+\)[^,]*),(.*)\)/)
  if (arr) {
    const index = arr[1].lastIndexOf(' ')
    const p = arr[1].slice(index).trim().replace('%', '') / 100
    const color1 = Color(arr[1].slice(0, index).trim())
    const color2 = Color(arr[2].trim())
    // 使用 color.js 的 mix 方法提前混出默认值
    return color2.mix(color1, p).hex()
  } else {
    return null
  }
}

module.exports.postcss = true
```

然后，再将 `postcss-plugin.js` 引入到 `vue.config.js` 中使用。

```js
// vue.config.js
const plugin = require('./postcss-plugin')

css: {
	loaderOptions: {
	  postcss: {
		postcssOptions: {
		  plugins: [plugin]
		}
	  }
	}
}
```

## 方案二：根据不同环境变量使用不同的scss主题文件

这个方案的基本思路就是通过环境变量动态生成 `alias` 的`@theme` 路径，从而使用不同的scss主题文件。

下面是具体的实现步骤：

首先，创建环境变量文件 `.env.development`， 并配置一下的环境变量 `VUE_APP_THEME`。

```js
VUE_APP_THEME = dev-theme
```

然后，创建对应的scss变量文件： `src\assets\themes\dev-theme\element-variables.scss` 。

```scss
/* 改变主题色变量 */
$--color-white: #fff !default;
$--color-primary: pink !default;
$--color-primary-light-1: mix($--color-white, $--color-primary, 10%) !default; /* 53a8ff */
$--color-primary-light-2: mix($--color-white, $--color-primary, 20%) !default; /* 66b1ff */
$--color-primary-light-3: mix($--color-white, $--color-primary, 30%) !default; /* 79bbff */
$--color-primary-light-4: mix($--color-white, $--color-primary, 40%) !default; /* 8cc5ff */
$--color-primary-light-5: mix($--color-white, $--color-primary, 50%) !default; /* a0cfff */
$--color-primary-light-6: mix($--color-white, $--color-primary, 60%) !default; /* b3d8ff */
$--color-primary-light-7: mix($--color-white, $--color-primary, 70%) !default; /* c6e2ff */
$--color-primary-light-8: mix($--color-white, $--color-primary, 80%) !default; /* d9ecff */
$--color-primary-light-9: mix($--color-white, $--color-primary, 90%) !default; /* ecf5ff */

$--color-success: #67c23a !default;
$--color-warning: #e6a23c !default;
$--color-danger: #f56c6c !default;
$--color-info: #909399 !default;

// $--color-primary: #409eff;
/* 改变 icon 字体路径变量，必需 */
$--font-path: "~element-ui/lib/theme-chalk/fonts";
@import "~element-ui/packages/theme-chalk/src/index";
```

然后，在`vue.config.js`中动态配置主题目录。

```js
// vue.config.js
resolve: {
  alias: {
	'@theme': path.resolve(__dirname, `src/assets/themes/${process.env.VUE_APP_THEME}`)
  }
}
```

在需要使用主题变量的文件中使用`@theme`引入变量。

```html
<style lang="scss">
@import '@theme/element-variables';

.el-button {
  background: $--color-primary;
}
</style>
```

最后，再在`package.json` 文件中配置打包方式。

```json
  {
    "build": "vue-cli-service build --mode development",
    "build:stage": "vue-cli-service build --mode staging",
    "build:prod": "vue-cli-service build --mode production",
  },
```
