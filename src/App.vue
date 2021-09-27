<template>
  <div id="app">
    <div class="week-info">今天是星期{{ week }}，{{ chickenSoup }}</div>
    <div v-if="isWorkTime" class="off-work">距离下班还有{{ offWork.hours }}小时{{ offWork.minutes }}分钟{{ offWork.seconds }}秒</div>
    <div v-else class="off-work">下班啦，回家陪陪家人吧</div>
    <div class="weekend">距离周末还有{{ weekend }}天{{ offWork.hours }}小时{{ offWork.minutes }}分钟{{ offWork.seconds }}秒</div>
    <div class="pay-off">距离发工资还有{{ payOff }}天</div>
  </div>
</template>

<script>
import { getChickenSoup } from './chickenSoup'
import { weekEnum } from './enum'
import moment from 'moment'
export default {
  name: 'App',
  data() {
    return {
      week: weekEnum[new moment().format('d')], //今天周几
      chickenSoup: getChickenSoup(), //一句毒鸡汤
      offWork: {}, //下班倒计时

      weekend: 0, // 距离放假的天数
      payOff: 0, //距离发工资的天数

      isWorkTime: true, //是否工作时间
      isWorkDay: true, //是否工作日
      timeer: null //页面定时器
    }
  },
  created() {
    this.timeer = setInterval(() => {
      this.setOffWork()
    }, 1000)
    this.setOffWork()
    this.setWeekend()
    this.setPayOff()
  },
  beforeDestroy() {
    clearInterval(this.timeer)
  },
  methods: {
    //设置下班倒计时
    setOffWork() {
      const now = new moment()
      const offWorkTime = new moment()

      if (offWorkTime.hours() >= 9 && offWorkTime.hours() < 18) {
        this.isWorkTime = true //工作时间
      } else {
        this.isWorkTime = false //非工作时间
      }

      if (offWorkTime.hours() >= 18) {
        offWorkTime.date(offWorkTime.date() + 1)
      }

      offWorkTime.hours(18)
      offWorkTime.minute(0)
      offWorkTime.second(0)
      offWorkTime.millisecond(0)
      this.offWork = moment.duration(offWorkTime.diff(now))._data
    },

    //设置距离周末天数
    setWeekend() {
      const now = new moment()
      const weekend = new moment()
      let week = weekend.day()
      if (week != 6) {
        week = 5 - week
      }
      //设置日期为下一个周五
      weekend.date(weekend.date() + week)
      this.weekend = weekend.diff(now, 'days')
    },

    //设置距离发工资天数
    setPayOff() {
      const now = new moment()
      const payOff = new moment()
      //设置日期为下一个10号
      if (payOff.date() > 10) {
        payOff.month(payOff.month() + 1)
      }
      payOff.date(10)
      this.payOff = payOff.diff(now, 'days')
    }
  }
}
</script>
<style lang="scss" scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  width: 500px;
  padding: 20px;
  .week-info {
    font-size: 16px;
    border-left: 5px solid #eee;
    padding: 5px 0 5px 20px;
    line-height: 22px;
    margin-bottom: 10px;
  }
  .off-work {
    padding: 10px;
    font-size: 14px;
  }
  .weekend {
    padding: 10px;
    font-size: 14px;
  }
  .pay-off {
    padding: 10px;
    font-size: 14px;
  }
}
</style>
