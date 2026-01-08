<template>
  <view class="xz-calendar">
    <view class="calendar_info">
      <picker mode="date" fields="month" :value="currentMonthInfo.date" @change="selectMonth">
        <view class="screen">
          <text class="year">{{ selectDayInfo.year }}</text>
          |
          <text class="month">{{ selectDayInfo.month }}.{{ selectDayInfo.day }}</text>
        </view>
      </picker>
      <view class="today" @click="selectDay(currentDayInfo, 'total')">今日</view>
    </view>
    <!-- 日期周 -->
    <view class="weeks">
      <view class="week" v-for="(week, index) in weeks" :key="week.value">
        {{ week.text }}
      </view>
    </view>
    <!-- 日期内容 -->
    <view class="days">
      <view class="currentMonth" v-if="type != 'week'">
        {{ currentMonthInfo.month }}
      </view>
      <view
        class="dayInfo"
        :class="{ current: currentDayInfo.date === item.date, select: selectDayInfo.date === item.date, noMonth: currentMonthInfo.month != item.month }"
        v-for="item in monthDaysInfo"
        :key="`${item.date}_${item.key}`"
        @click="selectDay(item, 'day')"
      >
        <view class="day">{{ item.day }}</view>
        <view class="mark" v-if="item.data && item.data.mark" :style="{ background: item.data.markColor ? item.data.markColor : '#2dd4aa' }"></view>
        <view class="info" v-if="item.data && item.data.info" :style="{ color: item.data.infoColor ? item.data.infoColor : '#2dd4aa' }">{{ item.data.info }}</view>
      </view>
    </view>
  </view>
</template>

<script>
/**
 * xzCalendar 自定义日历
 * @tutorial https://ext.dcloud.net.cn/plugin?id=25608
 * @description 实现展示一周或者一个月的日历
 * @property {Array} selected 打点数据，必填，格式：[{date:'2025-10-25',info:'描述',mark:true,markColor:'red',infoColor:'red',...}]
 * @property {String} type 日历类型，默认：month
 * 	@value month 月
 * 	@value week 周
 * @event {Function} changeDate 当前选择日期
 */
export default {
  props: {
    type: {
      type: String,
      default: 'month',
      validator: function (value) {
        return ['week', 'month'].includes(value);
      }
    },
    selected: {
      type: Array,
      required: false
    }
  },
  emits: ['changeDate', 'changeMonth'],
  data() {
    return {
      // 今日日期
      currentDayInfo: {},
      // 选择日期
      selectDayInfo: {},
      // 当前月份信息
      currentMonthInfo: {},
      // 日历头
      weeks: [
        { text: '周一', value: 1 },
        { text: '周二', value: 2 },
        { text: '周三', value: 3 },
        { text: '周四', value: 4 },
        { text: '周五', value: 5 },
        { text: '周六', value: 6 },
        { text: '周日', value: 0 }
      ],
      // 日历内容
      monthDaysInfo: []
    };
  },
  created() {
    this.initCurrentDay();
    let { year, month } = this.currentDayInfo;
    let monthDaysInfo = this.getDisplayDateInfo(year, month);
    this.monthDaysInfo = monthDaysInfo;
  },
  watch: {
    type: {
      handler: function (newVal, oldVal) {
        let { year, month } = this.currentMonthInfo;
        let monthDaysInfo = this.getDisplayDateInfo(year, month);
        this.monthDaysInfo = monthDaysInfo;
      }
    },
    selected: {
      handler: function (newVal, oldVal) {
        let { year, month } = this.currentMonthInfo;
        let monthDaysInfo = this.getDisplayDateInfo(year, month);
        this.monthDaysInfo = monthDaysInfo;
      },
      deep: true
    }
  },
  methods: {
    // 选择某一月
    selectMonth(info) {
      let date = info.detail.value.split('-');
      let year = Number(date[0]);
      let month = Number(date[1]);
      this.currentMonthInfo = {
        year,
        month,
        date: info.detail.value
      };
      let monthDaysInfo = this.getDisplayDateInfo(year, month);
      this.monthDaysInfo = monthDaysInfo;
      this.$emit('changeMonth', this.currentMonthInfo);
    },
    // 选择某一天
    selectDay(info, type) {
      if (info.month != this.currentMonthInfo.month) {
        let { year, month } = info;
        this.$emit('changeMonth', info);
        let monthDaysInfo = this.getDisplayDateInfo(year, month, type);
        this.monthDaysInfo = monthDaysInfo;
      }
      this.currentMonthInfo = info;
      this.selectDayInfo = info;
      this.$emit('changeDate', info);
    },
    // 获取需要展示的日期信息
    getDisplayDateInfo(year, month, type = 'day') {
      let monthDaysInfo = this.initMonthDate(year, month);
      if (!monthDaysInfo || monthDaysInfo.length <= 0) return [];
      if (this.$props.type !== 'week') {
        let newMonthDaysInfo = monthDaysInfo;
        if (this.$props.selected && this.$props.selected.length > 0) {
          newMonthDaysInfo = monthDaysInfo.map((day) => {
            return {
              ...day,
              data: this.$props.selected.find((item) => item.date === day.date)
            };
          });
        }
        return newMonthDaysInfo;
      }
      let displayIndex = 0;
      let sliceMonthDaysInfo = this.chunkArray(monthDaysInfo, 7);
      for (var index = 0; index < sliceMonthDaysInfo.length; index++) {
        var element = sliceMonthDaysInfo[index];
        let elementIndex = element.findIndex((item) => item.date === (type === 'total' ? this.currentDayInfo.date : this.selectDayInfo.date));
        if (elementIndex != -1) {
          displayIndex = index;
          break;
        }
      }
      let newMonthDaysInfo = sliceMonthDaysInfo[displayIndex];
      if (this.$props.selected && this.$props.selected.length > 0) {
        newMonthDaysInfo = sliceMonthDaysInfo[displayIndex].map((day) => {
          return {
            ...day,
            data: this.$props.selected.find((item) => item.date === day.date)
          };
        });
      }
      return newMonthDaysInfo;
    },
    // 初始化某个月信息
    initMonthDate(year, month) {
      // 当前月信息
      const currentMonthInfo = this.getMonthInfo(year, month);
      let monthDaysInfo = [];
      // 如果当前月不是周一则向前补上月
      if (currentMonthInfo.firstDayInWeek !== 1) {
        const prevMonthDate = new Date(year, month - 2, 1);
        const prevYear = prevMonthDate.getFullYear();
        const prevMonth = prevMonthDate.getMonth() + 1;
        const prevMonthInfo = this.getMonthInfo(prevYear, prevMonth);
        // 计算需要补多少天
        let prevDaysToShow;
        if (currentMonthInfo.firstDayInWeek === 0) prevDaysToShow = 6;
        else prevDaysToShow = currentMonthInfo.firstDayInWeek - 1;
        for (let i = prevMonthInfo.daysInMonth - prevDaysToShow + 1; i <= prevMonthInfo.daysInMonth; i++) {
          monthDaysInfo.push({ year: prevYear, month: prevMonth, day: i, date: `${prevYear}-${prevMonth}-${i}`, key: Date.now() });
        }
      }
      // 当前月
      for (let day = 1; day <= currentMonthInfo.daysInMonth; day++) {
        monthDaysInfo.push({ year, month, day, date: `${year}-${month}-${day}`, key: Date.now() });
      }
      // 如果当前月不是周日则向后补下月
      if (currentMonthInfo.lastDayInWeek !== 0) {
        const nextMonthDate = new Date(year, month, 1);
        const nextYear = nextMonthDate.getFullYear();
        const nextMonth = nextMonthDate.getMonth() + 1;
        const nextDaysToShow = 7 - currentMonthInfo.lastDayInWeek;
        for (let i = 1; i <= nextDaysToShow; i++) {
          monthDaysInfo.push({ year: nextYear, month: nextMonth, day: i, date: `${nextYear}-${nextMonth}-${i}`, key: Date.now() });
        }
      }
      return monthDaysInfo;
    },
    // 获取某个月部分信息
    getMonthInfo(year, month) {
      const daysInMonth = new Date(year, month, 0).getDate();
      const firstDayInWeek = new Date(year, month - 1, 1).getDay();
      const lastDayInWeek = new Date(year, month - 1, daysInMonth).getDay();
      return { daysInMonth, firstDayInWeek, lastDayInWeek };
    },
    // 将数组分7份
    chunkArray(array, chunkSize) {
      let chunks = [];
      for (let i = 0; i < array.length; i += chunkSize) {
        chunks.push(array.slice(i, i + chunkSize));
      }
      return chunks;
    },
    // 初始化当日信息
    initCurrentDay() {
      const now = new Date();
      const year = now.getFullYear();
      const month = now.getMonth() + 1;
      const day = now.getDate();
      let currentDayInfo = {
        year,
        month,
        day,
        date: `${year}-${month}-${day}`
      };
      this.currentMonthInfo = { year, month, date: `${year}-${month}` };
      this.currentDayInfo = currentDayInfo;
      this.selectDayInfo = currentDayInfo;
    }
  }
};
</script>

<style scoped>
.xz-calendar .calendar_info {
  display: flex;
  justify-content: space-between;
  padding: 0 20rpx;
  color: #999999;
}
.xz-calendar .calendar_info .screen {
  display: flex;
  align-items: center;
  gap: 10rpx;
}
.xz-calendar .calendar_info .year,
.xz-calendar .calendar_info .month {
  font-size: 26rpx;
  color: #000000;
}
.xz-calendar .calendar_info .today {
  font-size: 24rpx;
  color: #4aa5fa;
  background-color: #eaeaea;
  padding: 6rpx 30rpx;
  border-radius: 100rpx;
}
.xz-calendar .weeks {
  display: flex;
}
.xz-calendar .weeks .week {
  flex: 1;
  text-align: center;
  font-size: 26rpx;
  color: #999999;
  padding: 10rpx 0;
}
.xz-calendar .days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  position: relative;
}
.xz-calendar .days .currentMonth {
  position: absolute;
  font-size: 300rpx;
  color: #eaeaea;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
.xz-calendar .days .dayInfo {
  height: 90rpx;
  text-align: center;
  font-size: 26rpx;
  position: relative;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.xz-calendar .days .dayInfo .mark {
  position: absolute;
  width: 10rpx;
  height: 10rpx;
  border-radius: 10rpx;
  right: 15rpx;
  top: 15rpx;
}
.xz-calendar .days .dayInfo .info {
  font-size: 20rpx;
  text-align: center;
}
.xz-calendar .days .dayInfo.current {
  color: #5279ff;
}
.xz-calendar .days .dayInfo.noMonth {
  background: #f6f6f6;
  color: #999999;
}
.xz-calendar .days .dayInfo.select {
  background-color: #5279ff;
  color: #ffffff;
  border-radius: 10rpx;
}
</style>
