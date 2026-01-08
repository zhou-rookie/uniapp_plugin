<script lang="ts" setup></script>

<template>
  <view class="page">
    <view class="guide_1">盒子1</view>
    <view class="guide_2">盒子2</view>
    <xz-guide></xz-guide>
  </view>
</template>

<style lang="scss" scoped>
.guide_1 {
  width: 200rpx;
  height: 200rpx;
  border-radius: 20rpx;
  background-color: #00b26a;
  margin-top: 100rpx;
  margin-left: 100rpx;
}
.guide_2 {
  position: absolute;
  top: 500rpx;
  left: 300rpx;
  width: 200rpx;
  height: 200rpx;
  border-radius: 20rpx;
  background-color: #424242;
}
</style>
