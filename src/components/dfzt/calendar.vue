<template>
  <div  class="calendar">
    <div class="now-mouth">{{ displayDate }}</div>
    <div class="calendar_sd">
      <div class="calendar-title">
        <div class="calendar-select">
          <div v-for="(week,index) in weekName " :key="index" class="calendar-select-row">
            <input
              :id="'calendarSet_calendar_week_input_'+genID()"
              :ref="'week_'+index"
              class="checkbox"
              type="checkbox"
              :name="week"
              @change="checked('week',index)"
            >
          </div>
          <div class="calendar-select-row" />
        </div>
        <div class="calendar-select">
          <div v-for="(week, index) in weekName" :key="index" class="calendar-select-row">
            <div class="week"> {{ week }}</div>
          </div>
          <div class="calendar-select-row">
            <input :id="'calendarSet_calendar_week_input_'+genID()" ref="checkAll" class="checkbox" type="checkbox" @change="checkAll">
          </div>
        </div>
      </div>
      <div class="calendar-body">
        <div v-for="(week, weeKIndex) in calendarDay" :key="weeKIndex">
          <div class="calendar-select">
            <div
              v-for="(day, dayIndex) in week"
              :key="dayIndex"
              class="calendar-select-row"
              :class="{isLastChecked: test(day), nnn:testnnn(day),isSelected: day.select}"
              @click="selectDay(day)"
            >
              <div v-show="day.day" class="day">
                {{ day.day }}
              </div>
            </div>
            <div class="calendar-select-row">
              <input
                v-show="week[0].day || week[6].day"
                :id="'calendarSet_calendar_week_input_'+genID()"
                :ref="'row_'+weeKIndex"
                class="checkbox"
                type="checkbox"
                @change="checked('row',weeKIndex)"
              >
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
/**
*  Canlendar Module
* ----------------------
* Author:zoboy
* Date: 2019.06.12
*/

import moment from 'moment'
export default {
  name: 'Canlendar',
  components: {},
  props: {
    lastChecked: {
      type: Array,
      required: true,default:()=> new Array()
    },
    nnn:{
      type:String,
      required: true
    },
    mouth: {
      type: String,
      required: true
    },
    calendarOptiondDay: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      displayDate: '',
      nowDay: '',
      calendarDay: [],
      checkedDay: [],
      weekName: ['日', '一', '二', '三', '四', '五', '六']
    }
  },
  computed: {},
  watch: {
    mouth(newMouth) {
      this.createCalendar(newMouth)
    },
    calendarOptiondDay(val) {
      if (val) {
        this.createCalendar(this.mouth)
      }
    },
    calendarDay: {
      handler(canlendar) {
        this.checkedDay = []
        let row=new Set()
        let column=new Set()
        canlendar.forEach(week => {
          week.forEach(day => {
            if (day && day.day) {
              if (day.select) this.checkedDay.push(day)
              else{
                row.add(day.week)
                column.add(day.row)
              }
            }
          })
        })
        let week=[0,1,2,3,4,5,6]
        week.forEach(r=>{
            let weekRef = 'week_' + r
            let rowRef = 'row_' + r
            if(this.$refs[weekRef])
              if(row.has(r)) this.$refs[weekRef][0].checked = false
              else this.$refs[weekRef][0].checked = true
            if(this.$refs[rowRef])
              if(column.has(r)) this.$refs[rowRef][0].checked = false
              else this.$refs[rowRef][0].checked = true
          })
        // console.log(row,column)
        if(row && row.size>0 && column && column.size>0){
          this.$refs.checkAll.checked = false
          this.$emit('checkedAll', { 'mouth': this.mouth, 'checkedAll': false })
        }else{
          this.$refs.checkAll.checked = true
          this.$emit('checkedAll', { 'mouth': this.mouth, 'checkedAll': true })
        }
        this.$emit('checkedDate', { 'mouth': this.mouth, 'checkedDay': this.checkedDay })
        // console.log(this.checkedDay)
      },
      deep: true
    }
  },
  created() {
    this.createCalendar(this.mouth)
  },
  methods: {
    testnnn(day){
        return this.nnn === day.date
    },

    test(day){
      // console.log(day)
      return this.lastChecked.includes(day.date);
    },
    selectDay(day) {
      if (day && day.day) this.$emit('selectDay', day)
    },
    checked(target, index) {
      const checkRef = target + '_' + index
      const check = this.$refs[checkRef][0].checked
      this.calendarDay.forEach(week => {
        week.forEach(day => {
          if (day && day.day && day[target] === index) day.select = check
        })
      })
    },
    checkWork(isOnWork, select) {
      this.calendarDay.forEach(week => {
        week.forEach(day => {
          if (day && day.day) {
            if (isOnWork && (day.week > 0 && day.week < 6)) {
              day.select = select
            } else if (!isOnWork && (day.week === 0 || day.week === 6)) {
              day.select = select
            }
          }
        })
      })
    },
    checkAll() {
      const check = this.$refs.checkAll.checked
      this.calendarDay.forEach(week => {
        week.forEach(day => {
            if (day && day.day)  day.select = check
        })
      })
    },
    createCalendar(mouth) {
      if (mouth) {
        this.nowDay = moment(mouth).format('YYYY-MM')
      }else {
        this.nowDay = moment().format('YYYY-MM')
      }
      this.displayDate = moment(this.nowDay).format('YYYY年MM月')
      this.calendarDay = this.getCurrMonthDays(this.nowDay)
    },
    getCurrMonthDays(theDay) {
      // 获取当前月的第一天
      const start = moment(theDay).add('month', 0).format('YYYY-MM') + '-01'
      // 获取上一个月的总天数
      const lastMonthDays = moment(this.date).subtract(1, 'month').daysInMonth()
      // 构造日历的42个格子
      const daysArr = [[], [], [], [], [], []]
      // 获取当前月的第一天是星期几
      const currentWeekday = moment(start).format('E')
      // 获取当前月的总天数
      const nowMouthDays = moment(theDay).daysInMonth()
      for (let i = 0; i < 7; i++) { // 每一周
        // 虚拟天数
        let virtualDay
        if(currentWeekday-7===0) virtualDay=(lastMonthDays - currentWeekday) + i + 8
        else virtualDay=(lastMonthDays - currentWeekday) + i + 1
        for (let j = 0; j < daysArr.length; j++) { // 每一天
          daysArr[j][i] = this.getDay(i, j, start, virtualDay + (j * 7), lastMonthDays, nowMouthDays) // 每一行*7
        }
      }
      return daysArr
    },
    getDay(week, row, start, day, lastMonthDays, nowMouthDays) {
      let dayObj = {}
      if (day <= lastMonthDays) { // 上一月
        // return day
        dayObj = { 'day': '' }
      } else if (day <= (lastMonthDays + nowMouthDays)) {
        const dayNum = day - lastMonthDays
        const date = moment(start).add(dayNum - 1, 'days').format('YYYY-MM-DD')
        dayObj = {
          day: dayNum,
          date: date,
          week: week,
          row: row,
          isToday: new Date(date).getTime() === new Date(moment().format('YYYY-MM-DD')).getTime(),
          select: !!this.calendarOptiondDay.find((day) => day.date === date)
        }
      } else { // 下一月
        // return day - (lastMonthDays + nowMouthDays)
        dayObj = { 'day': '' }
      }
      return dayObj
    },
    genID() {
      return Number(Math.random().toString().substr(3, 4) + Date.now()).toString(36)
    }
  }
}
</script>

<style lang="scss" scoped>
.calendar{
  margin: 10px 5px;
  width: 248px;
  // height: 224px;
  display: flex;
  flex-direction: column;
  margin-bottom: 15px;
  margin-right: 15px;
  .calendar_sd{
  width: 241px;
  height: 209px;
  border-top: 1px solid #ddd;
  border-left: 1px solid #ddd;
  }
  .now-mouth{
    margin-bottom: 3px;
    font-weight: 549;
  }
  .calendar-title{
    .calendar-week{
      min-width: 35px;
    }
  }
  .calendar-select{
      display: flex;
      flex-direction: row;
      .calendar-select-row{
        min-width: 30px;
        min-height: 25px;
        border-right: 1px solid #ddd;
        border-bottom: 1px solid #ddd;
        // &:last-child {
        //   border-right: none;
        // }
        .checkbox{
          margin-top: 6px;
          margin-left: 7px;
          background-color: #ddd;
        }
        .week{
          margin-top: 6px;
          margin-left: 7px;
        }

      }
  }
  .calendar-body{
    .day {
      margin-top: 4px;
      margin-left: 9px;
      cursor:pointer;
    }
    .isLastChecked{
      background-color: #1890FF;
      color:#fff
    }
    .nnn{
      background-color: orange;
      color:#fff
    }
    .isSelected {
        background-color: #D2E5FE;
        color:#000
      }
    
  }
}
</style>
