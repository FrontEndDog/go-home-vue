<template>
  <div id="app">
    <div class="week-info">今天是星期{{ week }}，{{ chickenSoup }}</div>

    <div v-if="WorkStatus === 0" class="off-work">距离上班还有{{ onWork.hours }}小时{{ onWork.minutes }}分钟{{ onWork.seconds }}秒</div>
    <div v-if="WorkStatus === 1" class="off-work">距离下班还有{{ offWork.hours }}小时{{ offWork.minutes }}分钟{{ offWork.seconds }}秒</div>
    <div v-if="WorkStatus === 2" class="off-work">下班啦！快准备一下回家吧！</div>

    <div v-if="Object.keys(weekend).length" class="weekend">距离周末还有{{ weekend.days }}天{{ weekend.hours }}小时{{ weekend.minutes }}分钟{{ weekend.seconds }}秒</div>

    <div v-for="item in holidayList" :key="item.restDay[0]" class="weekend">距离{{ item.name }}假期还有{{ item.countDown.days }}天{{ item.countDown.hours }}小时{{ item.countDown.minutes }}分钟{{ item.countDown.seconds }}秒</div>

    <div class="pay-off">距离发工资还有{{ payOff }}天</div>

    <div class="flex date-time">
      <div>当前时间：{{ dateTime }}</div>
      <el-button size="mini" type="primary" @click="setClick">{{ setStatus ? '保存' : '设置' }}</el-button>
    </div>
    <!-- 设置面板 -->
    <SettingsPanel v-show="setStatus" ref="SettingsPanel" @updateConfig="setConfig"></SettingsPanel>
  </div>
</template>

<script>
import { getChickenSoup } from './chickenSoup'
import { weekEnum } from './enum'
import { getHourAndMinute } from './utils'
import holiday from './holiday'
import moment from 'moment'

export default {
  name: 'App',
  components: {
    SettingsPanel: () => import('./SettingsPanel')
  },
  data() {
    return {
      week: '', //今天周几
      chickenSoup: getChickenSoup(), //一句毒鸡汤

      onWork: {}, //上班倒计时
      offWork: {}, //下班倒计时

      weekend: null, // 距离周末的天数

      holidayList: [], //节假日倒计时，因为有多个节假日，所以是个数组

      payOff: null, //距离发工资的天数

      dateTime: '', //当前时间

      WorkStatus: null, //工作状态：0未开始上班，1正在上班，2已经下班
      isWorkDay: true, //是否工作日

      setStatus: false, //设置面板是否展开

      config: {}, //用户自定义的设置设置

      timeer: null //页面定时器
    }
  },
  created() {
    this.setConfig()
    this.update()
    this.timeer = setInterval(() => {
      this.update()
    }, 1000)
  },
  beforeDestroy() {
    //销毁定时器
    clearInterval(this.timeer)
  },
  methods: {
    //读取用户自定义设置
    setConfig() {
      const config = JSON.parse(localStorage.getItem('goHomeConfig') || '{}')
      this.config.onWorkTime = config.onWorkTime || '09:00'
      this.config.offWorkTime = config.offWorkTime || '18:00'
      this.config.workType = config.workType || 0
      this.config.payOffDay = config.payOffDay || 10
    },
    //每秒更新一次时间
    update() {
      this.setWorkStatus()
      this.setHoliday()
      this.setWeekend()
      this.setPayOff()
    },

    //设置工作状态
    setWorkStatus() {
      const now = new moment()
      const [onWorkHours, onWorkMinute] = getHourAndMinute(this.config.onWorkTime)
      const [offWorkHours, offWorkMinute] = getHourAndMinute(this.config.offWorkTime)

      if (now.hours() < onWorkHours || (now.hours() === onWorkHours && now.minute() < onWorkMinute)) {
        this.WorkStatus = 0 //未开始工作
        this.setOnWork() //设置上班倒计时
      } else if (now.hours() > offWorkHours || (now.hours() === offWorkHours && now.minute() >= offWorkMinute)) {
        this.WorkStatus = 2 //已下班
      } else {
        this.WorkStatus = 1 //工作中
        this.setOffWork() //设置下班倒计时
      }
      this.dateTime = now.format('YYYY-MM-DD HH:mm:ss')
      this.week = weekEnum[now.format('d')]
    },
    //上班倒计时
    setOnWork() {
      const now = new moment()
      const [onWorkHours, onWorkMinute] = getHourAndMinute(this.config.onWorkTime)
      const onWorkTime = new moment()
      onWorkTime.hours(onWorkHours)
      onWorkTime.minute(onWorkMinute)
      onWorkTime.second(0)
      onWorkTime.millisecond(0)
      this.onWork = moment.duration(onWorkTime.diff(now))._data
    },
    //下班倒计时
    setOffWork() {
      const now = new moment()
      const [offWorkHours, offWorkMinute] = getHourAndMinute(this.config.offWorkTime)
      const offWorkTime = new moment()
      offWorkTime.hours(offWorkHours)
      offWorkTime.minute(offWorkMinute)
      offWorkTime.second(0)
      offWorkTime.millisecond(0)
      this.offWork = moment.duration(offWorkTime.diff(now))._data
    },

    //设置距离节假日天数
    setHoliday() {
      const [offWorkHours, offWorkMinute] = getHourAndMinute(this.config.offWorkTime)
      this.holidayList = holiday
        .map(item => {
          const now = new moment()
          const firstDay = new moment(item.restDay[0])
          const yesterday = firstDay.date(firstDay.date() - 1)
          yesterday.hours(offWorkHours)
          yesterday.minute(offWorkMinute)
          yesterday.second(0)
          yesterday.millisecond(0)
          item.countDown = moment.duration(yesterday.diff(now))._data
          item.countDown.days = yesterday.diff(now, 'days')
          item.countDown.months = 0
          item.countDown.years = 0
          return item
        })
        .filter(item => item.countDown.days >= 0) //过滤已过期的节假日
        .filter((item, index) => index <= 2) //取前三条数据
    },

    //设置距离周末天数
    setWeekend() {
      const now = new moment()
      const weekend = new moment()
      //week是距离下一个周六的天数
      let week = weekend.day()
      if (week != 6) {
        week = 6 - week
      }
      //设置日期为下一个周六
      weekend.date(weekend.date() + week)

      //判断周六当天是否在调休或者节假日中
      const isInHoliday = this.holidayList.some(item => {
        const date = weekend.format('YYYY-MM-DD')
        return item.workDay.includes(date) || item.restDay.includes(date)
      })

      //如果本周六在调休或者节假日中，就不展示周末倒计时
      if (isInHoliday) {
        this.weekend = {}
      } else {
        const [offWorkHours, offWorkMinute] = getHourAndMinute(this.config.offWorkTime)
        weekend.date(weekend.date() - 1)
        weekend.hours(offWorkHours)
        weekend.minute(offWorkMinute)
        weekend.second(0)
        weekend.millisecond(0)
        this.weekend = moment.duration(weekend.diff(now))._data
      }
    },

    //设置距离发工资天数
    setPayOff() {
      const now = new moment()
      const payOff = new moment()

      //如果本月已经过了发薪日，月份往后加一个月
      if (payOff.date() > this.config.payOffDay) {
        payOff.month(payOff.month() + 1)
      }

      //将日期设为发薪日
      payOff.date(Math.min(this.config.payOffDay, payOff.daysInMonth()))

      //和现在相差天数
      this.payOff = payOff.diff(now, 'days')
    },
    async setClick() {
      if (this.setStatus) {
        await this.$refs.SettingsPanel.save()
      }
      this.setStatus = !this.setStatus
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
  .date-time {
    line-height: 28px;
    font-size: 14px;
    padding-left: 10px;
  }
}
</style>
