---
created: 2024-09-07 15:57
modified: 2024-09-07 15:57
Type: Entity
tags:
  - 编程/Vue
publish: true
date: 2024-09-11
path: tech/async-validator
categories:
  - 前端
description: 表单验证是非常常见的需求，常见的做法就是借助UI库来实现，像 AntDVue和ElementUI都自带表单验证，使用方法也很简单。但问题是如果我们项目中只需要进行验证而不需要输入框的红色边框，UI 组件是没有提供配置的。
title: 04. 如何不用ElementUI也能进行表单校验
Obsidian地址: obsidian://open?vault=content&file=C%20Knowledge%2F%E5%89%8D%E7%AB%AF%2F%E8%81%8C%E4%B8%9A%E8%A7%84%E5%88%92%2F%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8%2F%E5%85%AB%E8%82%A1%E6%96%87%2F04.%20%E5%A6%82%E4%BD%95%E4%B8%8D%E7%94%A8ElementUI%E4%B9%9F%E8%83%BD%E8%BF%9B%E8%A1%8C%E8%A1%A8%E5%8D%95%E6%A0%A1%E9%AA%8C.md
---
## 目录
## 写在前面

表单验证是非常常见的需求，常见的做法就是借助UI库来实现，像 [AntDVue](https://antdv.com/components/form-cn#components-form-demo-validate-static)和[ElementUI](https://element.eleme.cn/#/zh-CN/component/form) 都自带表单验证，使用方法也很简单。但问题是如果我们项目中只需要进行验证而不需要输入框的红色边框，UI 组件是没有提供配置的。

![如何不用ElementUI进行表单校验 20240910232648942](https://image.songxingguo.com/obsidian/20240911/%E5%A6%82%E4%BD%95%E4%B8%8D%E7%94%A8ElementUI%E8%BF%9B%E8%A1%8C%E8%A1%A8%E5%8D%95%E6%A0%A1%E9%AA%8C-20240910232648942.webp)

那有没有什么现成库能帮我们实现这样功能呢？当然有，在翻阅了[AntDVue](https://github.com/vueComponent/ant-design-vue/blob/main/components/form/utils/validateUtil.ts)和[ElementUI](https://github.com/ElemeFE/element/blob/dev/packages/form/src/form-item.vue)源码之后，我们会惊喜的发现，原来它们底层都使用一个叫[async-validator](https://github.com/yiminghe/async-validator)的表单验证库。既然UI框架都使用了，那想必这个库不会太差，并且当我们后续想再迁移到UI库我们也能很方便的实现，毕竟它们的数据结构都是一样的。当然除了 async-validator ，像 [validator.js](https://github.com/validatorjs/validator.js) 也是不错的表单验证库，不过有一定的学习成本。

下面👇我们就来说说怎么在项目中使用 async-validator 进行表单验证。

## 使用方法

使用方法很简单，只需要构建formRules 和 fromData 两个数据结构，然后将它们传递给  async-validator ，其中 `new AsyncValidator(this.formRules).validate(this.fromData)` 为最核心的代码。

```js
getFormItems() {
  const formItems = [];
  this.formConfig
  .filter(item => item.fieldname)
  .filter(item => item.type !== 'hidden' && !item.isHidden)
  .forEach(item => {
	formItems.push(FormItemSchema(item, formItems));
  });
  return formItems;
},
getFormRules(formItems) {
  const typeMap = {
	'selectinput': 'number',
	'addItem': 'array'
  }
  const rules = formItems
	.filter(item => item.require)
	.map(item => {
	  return [item.fieldKey, [{
		type: typeMap[item.type] || 'string',
		required: !!item.require,
		message: `${item.labelText.replace(/:|：/g, "")}不能为空`,
		trigger: 'blur'
	  }]]
	})
  return Object.fromEntries(rules);
},
getFormData(formItems) {
  const formData = formItems.map(item => [item.fieldKey, item.data]);
  return Object.fromEntries(formData);
},
validate() {
  const formItems = this.getFormItems();
  const formRules = this.getFormRules(formItems);
  const formData = this.getFormData(formItems);
  return new AsyncValidator(formRules).validate(formData);
},
```

这里定义了一个单项 FormItemSchema，将表单项标准化，并为每个表单项添加一个 fieldKey 避免表单项 `fieldname` 重复的问题。

```js
function FormItemSchema(item, formItems) {
  const require = item.isAddItem == undefined ? item.require : !item.isAddItem;
  return {
    ...item, 
    require,
    fieldKey: `${item.fieldname}${formItems.length}`
  }
}
```

最后，用 `try-catch` 块包裹 `await this.validate()` ，没有通过表单验证就进入 `catch` 块中，否则就提交表单。

```js
async submit({ operateType, auto, title, item }) {
  try {
	await this.validate();
	// 提交表单
	//...
  } catch ({ errors }) {
	if(!errors) return;
	const validateMessage = errors ? errors[0].message : '';
	this.$message.warning(validateMessage);
  }
},
```
