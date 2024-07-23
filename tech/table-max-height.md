---
created: 2024-07-23 22:52
modified: 2024-07-23 22:52
Type: Entity
tags:
  - 编程/Vue
publish: true
date: 2024-07-23 22:52
title: 计算页面中 el-table 表格最大高度max-height
description: 在工作中遇到了多个滚动条的情况，后来仔细研究了一下是因为el-table的max-height设置为固定值导致的，当窗口区域过小就会出现这种情况。
categories:
  - 前端
---
## 目录

## 写在前面

在工作中遇到了多个滚动条的情况，后来仔细研究了一下是因为el-table的max-height设置为固定值导致的，当窗口区域过小就会出现这种情况。

![计算页面中 el table 表格最大高度max height 20240722184637857](https://image.songxingguo.com/obsidian/20240723/%E8%AE%A1%E7%AE%97%E9%A1%B5%E9%9D%A2%E4%B8%AD%20el-table%20%E8%A1%A8%E6%A0%BC%E6%9C%80%E5%A4%A7%E9%AB%98%E5%BA%A6max-height-20240722184637857.jpeg)

知道问题了就可以对症下药，根据屏幕的大小动态设置max-heigth。在网上看了很多的方案都是减去除了表格以外其他元素的宽高来动态计算高度，显然这种方法不够健壮一旦新增或减少元素，我们就必须调整代码。于是我想到页面流的特性，可以用 `flex: 1` +  offsetHeight 的方式实现，`flex: 1` 始终会撑满剩下区域，那我们就可以获取到`flex: 1` 的容器高度设置给el-table从而实现动态高度，无需手动计算，这样既简单又健壮。

## Vue3写法

为了增加代码的可复用性，Vue3中可以将相关的代码封装成一个hooks。这段代码中最主要的功能是当窗口变化时获取到容器的 offsetHeight 设置给 maxHeight。

```js
  function useMaxWidth() {
	const tableRef = ref();
	const maxHeight = ref(0);

	onMounted(() => {
	  window.addEventListener("resize", resize);
	});

	onUnmounted(() => {
	  window.removeEventListener("resize", resize);
	});

	function resize() {
	  maxHeight.value = 0;
	  nextTick(() => {
		maxHeight.value = tableRef.value.offsetHeight;
	  });
	}
	return { maxHeight, tableRef, resize };
  }
```

接着，在setup 中使用 useMaxWidth， 在加载完数据之后，执行一次 `resize()` 来获取maxHeight， 并导出tableData、 tableRef 和 maxHeight。

```js
  const App = {
	setup() {
	  let tableData = ref([]);
	  onBeforeMount(async () => {
		tableData.value = await new Promise((resolve) => {
		  setTimeout(() => {
			const data = new Array(100).fill({
			  date: "2016-05-02",
			  name: "王小虎",
			  address: "上海市普陀区金沙江路 1518 弄",
			});
			resolve(data);
		  }, 2000);
		});
	  });

	  const { maxHeight, tableRef, resize } = useMaxWidth();

	  resize();

	  return {
		tableData,
		tableRef,
		maxHeight,
	  };
	},
  };
  const app = createApp(App);
  app.use(ElementPlus);
  app.mount("#app");
```

最后，在 el-table外定义了一个 `flex:1` 的容器，将 tableRef 绑定到上面，并将 tableData 和 maxHeight 绑定到 el-table 上。

```html
	<main class="HolyGrail-content" ref="tableRef">
	  <el-table :data="tableData" :max-height="maxHeight">
		<el-table-column prop="date" label="日期" sortable width="180">
		</el-table-column>
		<el-table-column prop="name" label="姓名" sortable width="180">
		</el-table-column>
		<el-table-column prop="address" label="地址" :formatter="formatter">
		</el-table-column>
	  </el-table>
	</main>
```

```css
.HolyGrail-content {
 flex: 1;
}
```

## Vue2写法

在 Vue3 中可以通过 hooks 复用代码，Vue2 则可以将功能封装成一个 table-container 组件，并将 el-table 插入其中。

```jsx
<template>
 <div class="table-container" ref="table">
  <slot :maxHeight="maxHeight"></slot>
 </div>
</template>
<script>
 export default {
   name:'',
   data () {
     return {
      maxHeight: 0
     }
   },
   mounted() {
    window.addEventListener('resize', this.resize);
   },
   beforeDestroy() {
    window.removeEventListener('resize', this.resize)
   },
   methods:{
    resize() {
      this.maxHeight = 0;
      this.$nextTick(() => {
        this.maxHeight = this.$refs.table.offsetHeight;
      })
    }
   },
 }
</script>
<style scoped>
  .table-container {
    height: 100%;
  } 
</style>
```

```jsx
  <table-container ref="tableContainer" v-slot="slotProps">
	  <el-table :data="tableList" :max-height="slotProps.maxHeight" >
	  </el-table>
  </table-container>
```

当然，也需要在接口数据加载完成的时候执行一下 `resize()` 方法。

```jsx
this.$refs.tableContainer.resize();
```

>  
> 重新设置max-height表格会重新计算， 会导致 scrollTop 会重置为0，如何实现上拉加载，下拉刷新！|下拉加载的滚动条会回到顶部，因此可以只在第一页计算最大高度。
> 
> ```js
>  if(this.listQuery.page === 1) this.$refs.tableContainer.resize();
> ```
> 

