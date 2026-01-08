<template>
  <view class="guide"></view>
</template>

<script>
export default {
  data() {
    return {};
  },
  mounted() {
    const query = uni.createSelectorQuery().in(this);
    query
      .select('.guide_1')
      .boundingClientRect((data) => {
        console.log('得到布局位置信息', data);
      })
      .exec();
  }
};
</script>

<style scoped></style>