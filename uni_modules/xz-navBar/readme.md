# xz-navBar 自定义导航栏
> 自定义导航栏

### Props
| 属性名			| 类型	| 是否必填	|默认	|可选									|说明				|
|-------			|-------|-------	|-------|-------								|-------			|
| title				| String|false		|''		|无										|导航栏标题			|
| titleAlign		| String|false		|center	|center(居中)、left(左边)、right(右边)	|导航栏标题对齐方式	|
| navBarStyle		| Object|false		|{}		|无										|导航栏自定义样式		|
| navBarBottomType	| String|false		|border	|border(边框)、shadow(阴影)				|navBar下边框显示类型	|

### Emits
| 事件名		| 接收值	|说明				|
|-------		|-------|-------			|
|getNavHeight	|height	|接收navBar元素的高度	|

### Slots
| 插槽		|说明		|
|----		|----		|
|navBarLeft	|导航栏左边	|
|navBarRight|导航栏右边	|

### 代码示例

#### 一级导航样式
```js
const navBarStyle = {
	filter: 'none',
	position: 'relative',
	backgroundColor: 'transparent'
};
```
```vue
<xz-navBar title="背景透明，导航占据空间" titleAlign="left" :navBarStyle="navBarStyle"></xz-navBar>
<xz-navBar title="普通" :navBarStyle="navBarStyle"></xz-navBar>
```

#### 二级导航样式
```vue
<xz-navBar title="二级导航" :navBarStyle="navBarStyle">
	<template #navBarLeft>
		<view class="left">
			<uni-icons type="left" size="28"></uni-icons>
		</view>
	</template>
</xz-navBar>
```

#### 自定义样式
```vue
<xz-navBar>
	<template #navBarLeft>
		<view class="city">北京</view>
	</template>
	<template #navBarRight>
		<view class="input-view">
			<uni-icons class="input-uni-icon" type="search" size="18" color="#999" />
			<input confirm-type="search" class="nav-bar-input" type="text" placeholder="输入搜索关键词" @confirm="confirm" />
		</view>
	</template>
</xz-navBar>
```
```css
.city {
	/* #ifndef APP-PLUS-NVUE */
	display: flex;
	/* #endif */
	flex-direction: row;
	align-items: center;
	justify-content: flex-start;
	// width: 160rpx;
	margin-left: 4px;
	margin-right: 10px;
}

.input-view {
	/* #ifndef APP-PLUS-NVUE */
	display: flex;
	/* #endif */
	flex-direction: row;
	// width: 500rpx;
	flex: 1;
	background-color: #f8f8f8;
	height: 30px;
	border-radius: 15px;
	padding: 0 15px;
	flex-wrap: nowrap;
	margin: 7px 0;
	line-height: 30px;
}

.input-uni-icon {
	line-height: 30px;
}

.nav-bar-input {
	height: 30px;
	line-height: 30px;
	/* #ifdef APP-PLUS-NVUE */
	width: 370rpx;
	/* #endif */
	padding: 0 5px;
	font-size: 12px;
	background-color: #f8f8f8;
}
```

### 注意事项：
1. 需要在pages.json页面的导航栏样式设置为custom
```json
{
	"path": "pages/index/index",
	"style": {
		"navigationBarTitleText": "首页",
		"navigationStyle": "custom"
	}
},
```
2. 使用自定义navBar会遮住页面顶部一个navBar的高度，需要页面添加上边距或者使用scroll-view组件更好的完成页面效果。

### scroll-view配合自定义tabBar以及自定义导航
- xz-tabBar：[查看](https://ext.dcloud.net.cn/plugin?id=20350)
- xz-navBar：[查看](https://ext.dcloud.net.cn/plugin?id=25024)

```vue
<script lang="ts" setup>
import { onBeforeMount, ref, watch } from 'vue';

/**
 * @property {String} title 页面标题
 * @property {String} pageKey 页面key
 * @property {String} navBarHeight 如果用插槽自定义导航栏则需要传入其高度
 * @property {String} background 页面背景颜色
 */
interface Prop {
	title: string;
	pageKey: string;
	navBarHeight: string;
	background: string;
}
const props = withDefaults(defineProps<Prop>(), {});

// 隐藏tabBar
onBeforeMount(() => {
	uni.hideTabBar();
});

const list = [
	{
		text: '首页',
		pageKey: 'calendar',
		pagePath: '/pages/home/index',
		iconPath: '/static/tabBar/home.png',
		selectedIconPath: '/static/tabBar/home.png'
	},
	{
		text: '我的',
		pageKey: 'user',
		pagePath: '/pages/user/index',
		iconPath: '/static/tabBar/user.png',
		selectedIconPath: '/static/tabBar/user_select.png'
	}
];

const openPage = (url: string) => {
	uni.switchTab({
		url
	});
};

// 计算页面高度
let scrollHeight = ref<number>(0);
let navBarHeight = ref<number>(0);
let tabBarHeight = ref<number>(0);
const getNavHeight = (height: number) => {
	navBarHeight.value = height;
};
const getTabHeight = (height: number) => {
	tabBarHeight.value = height;
};

watch(
	() => props.navBarHeight,
	(newVal) => {
		if (newVal != '0') navBarHeight.value = Number(newVal);
	},
	{
		deep: true,
		immediate: true
	}
);
</script>

<template>
	<view class="page_common" :style="{ background }">
		<!-- navBar -->
		<slot name="navBar" v-if="$slots.navBar"></slot>
		<xz-navBar v-else :navBarStyle="{ position: 'relative' }" :title="props.title" @getNavHeight="getNavHeight"></xz-navBar>
		<!-- 主体内容 -->
		<scroll-view scroll-y :style="{ height: `calc(100vh - ${navBarHeight + tabBarHeight}px)` }">
			<view class="page_content">
				<slot></slot>
			</view>
		</scroll-view>
		<!-- tabBar -->
		<xz-tabBar
			:tabBarStyle="{ position: 'relative' }"
			:list="list"
			:pageKey="props.pageKey"
			color="#444444"
			selectColor="#00B386"
			tabBarTopType="shadow"
			@openPage="openPage"
			@getTabHeight="getTabHeight"
		></xz-tabBar>
	</view>
</template>

<style lang="scss" scoped>
.page_common {
	height: 100vh;
	overflow: hidden;
	background-color: rgba(238, 238, 238, 0.2);

	.page_content {
		overflow: hidden;
		box-sizing: border-box;
	}
}
</style>
```