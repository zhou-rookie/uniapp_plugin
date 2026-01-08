<template>
	<view class="tabBar" ref="tabBarRef" :style="{ ...tabBarStyle, ...safeAreaInfo }" :class="[tabBarTopType]">
		<view
			class="tabBar_item"
			:style="{ color: item.pageKey === pageKey ? selectColor : color, ...tabBarItemStyle }"
			v-for="item in leftList"
			:key="item.pageKey"
			@click="openPage(item)"
		>
			<image class="item_icon" :src="item.pageKey === pageKey ? item.selectedIconPath : item.iconPath"></image>
			<view class="item_text">{{ item.text }}</view>
		</view>
		<view class="tabBar_item" v-if="$slots.protrusion">
			<slot name="protrusion"></slot>
		</view>
		<view
			class="tabBar_item"
			:style="{ color: item.pageKey === pageKey ? selectColor : color, ...tabBarItemStyle }"
			v-for="item in rightList"
			:key="item.pageKey"
			@click="openPage(item)"
		>
			<image class="item_icon" :src="item.pageKey === pageKey ? item.selectedIconPath : item.iconPath"></image>
			<view class="item_text">{{ item.text }}</view>
		</view>
	</view>
</template>

<script>
/**
 * xzTabBar 自定义tabBar
 * @tutorial https://ext.dcloud.net.cn/plugin?id=20350
 * @description 实现隔离底部安全区域和通过插槽自定义凸起的tabBar
 * @property {Array} list tabBar数据，必填，格式：[{text:'名称',pageKey:'路由key',pagePath:'路由地址',iconPath:'默认icon',selectedIconPath:'选中icon'}]，pageKey必须唯一
 * @property {String} pageKey 当前tabBar唯一值，必填
 * @property {String} color 文字颜色，默认：#333333
 * @property {String} selectColor 选中文字颜色，默认：#ff0000
 * @property {String} tabBarTopType tabBar上边框显示类型，默认：border
 * 	@value border 边框
 * 	@value shadow 阴影
 * @property {Object} tabBarStyle tabBa自定义样式，格式{height: '100rpx'}
 * @property {Object} tabBarItemStyle tabBa每一项自定义样式，格式{height: '100rpx'}
 * @property {String} safeAreaShowType 安全区域类型，默认padding
 * 	@value padding 内边距
 * 	@value margin  外边距
 * @event {Function} openPage 点击tabBar每一项触发，返回跳转的路由地址
 * @event {Function} getTabHeight 获取tabBar高度，返回一个高度
 */
export default {
	props: {
		list: {
			type: Array,
			required: true
		},
		pageKey: {
			type: String,
			required: true
		},
		color: {
			type: String,
			default: '#333333'
		},
		selectColor: {
			type: String,
			default: '#ff0000'
		},
		tabBarTopType: {
			type: String,
			default: 'border'
		},
		tabBarStyle: {
			type: Object
		},
		tabBarItemStyle: {
			type: Object
		},
		safeAreaShowType: {
			type: String,
			default: 'padding'
		}
	},
	emits: ['openPage', 'getTabHeight'],
	data() {
		return {
			safeAreaInfo: {
				paddingBottom: 0,
				marginBottom: 0,
				bottom: 0
			},
			leftList: [],
			rightList: []
		};
	},
	created() {
		// 初始化配置：隐藏tabBar、获取底部安全区域、list分组
		this.initInfo();
	},
	mounted() {
		// 获取导航栏高度
		this.getTabBarHeight();
	},
	methods: {
		// 初始化配置
		initInfo() {
			let that = this;
			// 隐藏tabBar
			uni.hideTabBar();
			// 获取底部安全区域
			let getSystemInfo = uni.getSystemInfoSync();
			let bottom = (getSystemInfo.safeAreaInsets.bottom || 2) + 'px';
			if (that.$props.safeAreaShowType === 'padding') that.safeAreaInfo.paddingBottom = bottom;
			else that.safeAreaInfo.marginBottom = bottom;
			if (bottom === '2px' && that.$props.tabBarStyle && that.$props.tabBarStyle.bottom) that.safeAreaInfo.bottom = that.$props.tabBarStyle.bottom;
			// 分割数组
			if (that.$props.list && that.$props.list.length > 0) {
				let list = that.$props.list;
				that.leftList = list.slice(0, Math.ceil(list.length / 2));
				that.rightList = list.slice(Math.ceil(list.length / 2));
			}
		},
		// 跳转路由
		openPage(item) {
			let that = this;
			if (item.pageKey === that.$props.pageKey) return;
			let selectTabMethod = that.$props.selectTabMethod;
			let pagePath = item.pagePath.startsWith('/') ? item.pagePath : `/${item.pagePath}`;
			that.$emit('openPage', pagePath);
		},
		// 获取TabBar高度
		getTabBarHeight() {
			this.$nextTick(() => {
				// #ifdef MP
				const query = uni.createSelectorQuery().in(this);
				query
					.select('.tabBar')
					.boundingClientRect((rect) => {
						if (rect) this.$emit('getTabHeight', rect.height);
					})
					.exec();
				// #endif

				// #ifndef MP
				const el = this.$refs.tabBarRef.$el;
				if (el) this.$emit('getTabHeight', el.offsetHeight);
				// #endif
			});
		}
	}
};
</script>

<style scoped>
.tabBar {
	position: fixed;
	left: 0;
	right: 0;
	bottom: 0;
	z-index: 999;
	height: 100rpx;
	display: flex;
	align-items: center;
	justify-content: space-around;
	padding: 5rpx 0;
	background-color: #ffffff;
}
.tabBar.border {
	filter: drop-shadow(0 0 2rpx rgba(0, 0, 0, 0.5));
}
.tabBar.shadow {
	filter: drop-shadow(5rpx 5rpx 10rpx rgba(0, 0, 0, 0.2));
}
.tabBar .tabBar_item {
	height: 100%;
	flex: 1;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	flex-shrink: 0;
}
.tabBar_item .item_icon {
	display: block;
	width: 45rpx;
	height: 45rpx;
}
.tabBar_item .item_text {
	font-size: 22rpx;
}
</style>
