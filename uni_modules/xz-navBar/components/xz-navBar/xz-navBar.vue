<template>
  <view class="navBar" ref="navBarRef" :style="{ ...menuStyleInfo, ...navBarStyle }" :class="[navBarBottomType]">
    <slot name="navBarLeft" v-if="$slots.navBarLeft"></slot>
    <view class="title" v-if="title" :class="[titleAlign]">
      <text>{{ title }}</text>
    </view>
    <slot name="navBarRight" v-if="$slots.navBarRight"></slot>
  </view>
</template>

<script>
/**
 * xzNavBar 自定义导航栏
 * @tutorial https://ext.dcloud.net.cn/plugin?id=25024
 * @description 自定义导航栏
 * @property {String} title 导航栏标题
 * @property {String} titleAlign 导航栏标题对齐方式，默认：center
 * 	@value center 居中
 * 	@value left   左边
 * 	@value right  右边
 * @property {Object} navBarStyle 导航栏自定义样式
 * @property {String} navBarBottomType navBar下边框显示类型，默认：border
 * 	@value border 边框
 * 	@value shadow 阴影
 * @event {Function} getNavHeight 获取导航栏高度，返回一个高度
 */
export default {
  props: {
    title: {
      type: String
    },
    navBarStyle: {
      type: Object
    },
    navBarBottomType: {
      type: String,
      default: 'border'
    },
    titleAlign: {
      type: String,
      default: 'center'
    }
  },
  emits: ['getNavHeight'],
  data() {
    return {
      menuStyleInfo: {
        paddingTop: 0,
        height: 0,
        paddingRight: 0
      }
    };
  },
  created() {
    // 初始化配置：获取顶部安全区域
    this.initInfo();
  },
  mounted() {
    // 获取导航栏高度
    this.getNavBarHeight();
  },
  methods: {
    // 初始化配置
    initInfo() {
      let that = this;
      // 胶囊信息
      try {
        let menuButtonInfo = uni.getMenuButtonBoundingClientRect();
        that.menuStyleInfo.paddingTop = menuButtonInfo.top + 'px';
        that.menuStyleInfo.height = menuButtonInfo.height + 'px';
        let getSystemInfo = uni.getSystemInfoSync();
        that.menuStyleInfo.paddingRight = getSystemInfo.screenWidth - menuButtonInfo.right + menuButtonInfo.width + 'px';
      } catch {
        that.menuStyleInfo.height = '44px';
      }
    },
    // 获取导航栏高度
    getNavBarHeight() {
      // #ifdef MP
      const query = uni.createSelectorQuery().in(this);
      query
        .select('.navBar')
        .boundingClientRect((rect) => {
          if (rect) this.$emit('getNavHeight', rect.bottom);
        })
        .exec();
      // #endif
      // #ifndef MP
      const el = this.$refs.navBarRef.$el;
      if (el) this.$emit('getNavHeight', el.offsetHeight);
      // #endif
    }
  }
};
</script>

<style scoped>
.navBar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 999;
  padding-bottom: 10rpx;
  display: flex;
  align-items: center;
  background-color: #fff;
}
.navBar.border {
  filter: drop-shadow(0 0 2rpx rgba(0, 0, 0, 0.5));
}
.navBar.shadow {
  filter: drop-shadow(5rpx 5rpx 10rpx rgba(0, 0, 0, 0.2));
}
.navBar .title {
  flex: 1;
  position: relative;
  flex-shrink: 0;
}
.title.center {
  text-align: center;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}
.title.left {
  text-align: left;
}
.title.right {
  text-align: right;
}
</style>
