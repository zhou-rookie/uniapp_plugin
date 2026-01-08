# xz-tabBar  自定义tabBar
> 实现隔离底部安全区域和通过插槽自定义凸起的tabBar

### Props
| 属性名			| 类型	| 是否必填	|默认	|可选							|说明																																|
|-------			|-------|-------	|-------|-------						|-------																															|
| list				| Array	|true		|[]		|无								|tabBar数据；格式：[{text:'名称',pageKey:'路由key',pagePath:'路由地址',iconPath:'默认icon',selectedIconPath:'选中icon'}]；pageKey必须唯一	|
| pageKey			| String|true		|""		|无								|当前tabBar唯一值																														|
| safeAreaShowType	| String|true		|padding|padding(内边距)、margin(外边距)	|安全区域类型																															|
| tabBarTopType		| String|false		|border	|border(边框)、shadow(阴影)		|tabBar上边框显示类型																													|
| color				| String|false		|#333333|无								|文字颜色																															|
| selectColor		| String|false		|#ff0000|无								|选中文字颜色																															|
| tabBarStyle		| Object|false		|{}		|无								|tabBa自定义样式；格式{height: '100rpx'}																								|
| tabBarItemStyle	| Object|false		|{}		|无								|tabBa每一项自定义样式；格式{height: '100rpx'}																							|

### Emits
| 事件名		| 接收值	|说明						|
|-------		|-------|-------					|
|openPage		|url	|接收点击tabBar项的路由地址	|
|getTabHeight	|height	|接收tabBar元素的高度			|

### Slots
| 插槽		|说明							|
|----		|----							|
|protrusion	|中间凸起，请查看代码示例中凸起样式	|


### 代码示例

#### 常规数据
```js
const list = [
	{
		text: '首页',
		pageKey: 'index',
		pagePath: '/pages/index/index',
		iconPath: '/static/home.png',
		selectedIconPath: '/static/selectedHome.png'
	},
	{
		text: '我的',
		pageKey: 'user',
		pagePath: '/pages/user/index',
		iconPath: '/static/user.png',
		selectedIconPath: '/static/selectedUser.png'
	}
];

const openPage = (url) => {
	uni.switchTab({
		url
	});
};
```

#### 普通样式
```vue
<template>
	<xz-tabBar :list="list" pageKey="index" selectColor="#1296db" @openPage="openPage"></xz-tabBar>
</template>
```

#### 凸起样式
```vue
<template>
	<xz-tabBar tabBarTopType="shadow" :list="list" pageKey="index" selectColor="#1296db" @openPage="openPage">
		<template #protrusion>
			<view class="protrusion" @click="openPage('/pages/menu/index')">
				<image class="protrusion_img" src="/static/menu.png" mode=""></image>
			</view>
		</template>
	</xz-tabBar>
</template>
<style lang="less" scoped>
.protrusion {
	position: absolute;
	width: 130rpx;
	height: 130rpx;
	border-radius: 50%;
	top: -50rpx;
	background-color: #ffffff;
	display: flex;
	align-items: center;
	justify-content: center;
	.protrusion_img {
		width: 80%;
		height: 80%;
		border-radius: 50%;
	}
}
</style>

```

#### 自定义样式
```js
const tabBarStyle = {
	height: '100rpx',
	left: '20rpx',
	right: '20rpx',
	bottom: '20rpx',
	borderRadius: '10rpx'
};
```
```vue
<xz-tabBar :list="list" pageKey="user" safeAreaShowType="margin" :tabBarStyle="tabBarStyle" selectColor="#1296db" @openPage="openPage"></xz-tabBar>
```

### 注意事项：
1. 使用自定义tabBar会遮住页面底部一个tabBar的高度，需要页面添加下边距或者使用scroll-view组件更好的完成页面效果。
2. 虽然自定义tabBar添加了隐藏系统tabBar，但还是需要在App.vue自行添加uni.hideTabBar()，以提高体验感。
3. 可能会闪屏，如果建议自行解决

### 官方注意事项：自定义tabBar的性能体验会低于原生tabBar。App和小程序端非必要不要自定义。
- H5端的自定义tabBar组件：H5端不存在原生tabBar性能更高的概念，并且宽屏下常见的tabBar在顶部而不是底部，此时可以使用 custom-tab-bar组件来自定义
- 普通自定义tabBar：使用view自行绘制tabBar。如果页面是多页方式，切换tabBar将无法保持底部tabBar一直显示。所以这种情况建议使用单页方式。单页方式，如果是复杂页面，应用性能会下降明显，需减少页面复杂度。如果是App端，nvue单页的性能会显著高于vue页面
- 微信小程序自定义tabbar：微信提供一直基于webview自定义tabBar的方案。该功能体验不佳，不太推荐使用。
- 原生的tabbar有且只有一个且在首页。二级页如需的tab，需自行编写view来实现。一般二级页面更适合的导航是 segement组件


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