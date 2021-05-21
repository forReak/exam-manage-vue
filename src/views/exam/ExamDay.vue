<template>
  <div >
    <Card dis-hover>
      <div class="calendar">
        <div class="chose_sd star">
          当前年份<DatePicker v-model="year" disable size="small" type="year" placeholder="选择年" class="choseYear"  @on-change="changeYear" @on-blur="changeYear" :editable="false" :clearable="false" />
          <RadioGroup v-model="choseWorkDay" @on-change="changeWorkDay" style="margin-left: 2%;">
              <Radio label="工作日" ref="workDay"></Radio>
              <Radio label="非工作日" ref="noWorkDay"></Radio>
          </RadioGroup>
          <!-- <button @click="logDate">保存</button> -->
           <Checkbox v-model="choseCheckAll" @on-change="changeCheckAll">全选</Checkbox>
        </div>
        <div class="chose_sd star" style="margin-top:10px">
          <a-button @click="logDate" type="primary" icon="plus">设置考试日</a-button>
          <a-button style="margin-left:10px" @click="removeDay" type="primary" icon="delete">取消考试日</a-button>
        </div>


        <div class="chose_sd star" style="margin-top:10px">

            <span style="background-color:#1890FF">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span> &nbsp;已安排考试&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
            <span style="background-color:#ffffff;border:1px solid black">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span> &nbsp;可以安排考试&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
            <span style="background-color:orange">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span> &nbsp;今天&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        </div>
        <div class="row">
          <calendar
            v-for="(mouth,index) in mouths"
            :key="index"
            :ref="mouth"
            :mouth="mouth"
            :calendar-optiond-day="calendarOptiondDay"
            @checkedDate="checkedDate"
            @checkedAll="checkedAll"
            @selectDay="onClick"
            :lastChecked = lastChecked
            :nnn = nnn
          />
        </div>
      </div>
    </Card>
  </div>
</template>
<script>
    /**
   * dateList数据格式：["2019-01-05", "2019-01-06", "2019-01-12"]
   * 日期组件数据回填，需要的格式，为数组对象，如：
   * [{date: "2019-01-05",day: 6,isToday: false,row: 0,select: true,week: 6},{date: "2019-01-06",day: 6,isToday: false,row: 0,select: true,week: 6}]
   */
import Calendar from '@/components/dfzt/calendar'
import moment from 'moment'
import { httpAction, postAction,getAction } from '@/api/manage'
export default {
  name: 'CalendarDemo',
  components: { Calendar },
  data() {
    return {
      year: moment().format('YYYY'),
      choseWorkDay:'',
      choseCheckAll:false,
      mouths: [],
      checkedDates: [], // 最终选中的全部日期
      calendarOptiondDay: [],//选中日期数组
      allCheckDay: [],
      allSelectedDay: [],
      lastChecked:[],
      nnn:"",
      dayObj: {
        date: '',
        day: 6,
        isToday: false,
        row: 0,
        select: true,
        week: 6
      },
    }
  },
  watch: {

  },
  created(){
    this.checkedDates = [];
    this.getAllDay();
    this.createToday();
  },
  mounted() {
    this.$nextTick(() => {
      this.changeYear()
    })
  },
  methods: {
    createToday(){
      var date = new Date();
      var year = date.getFullYear();
      var month = date.getMonth() + 1;
      var day = date.getDate();
      if (month < 10) {
          month = "0" + month;
      }
      if (day < 10) {
          day = "0" +  day;
      }
      var nowDate = year + "-" + month + "-" + day;
      this.nnn = nowDate;
    },

    getAllDay(){
      
      getAction("/exam/bizExamDay/getAllDayList",{}).then((res)=>{
        if(res.success){
          this.lastChecked = res.result
          
          console.log(res.result)
        }else{
          alert(res.message);
        }
      });
        
    },
    removeDay(){
      let selectArr = this.checkedDates;
      let selectDate = [];
      selectArr.forEach( e => selectDate.push(e.date));
      if(selectDate.length>0){
        let params = {examDayList:selectDate};
        postAction("/exam/bizExamDay/removeDay",params).then((res)=>{
          if(res.success){
            alert(res.message);
          }else{
            alert(res.message);
          }
        });
      }
    },
    logDate(){
      let selectArr = this.checkedDates;
      let selectDate = [];
      selectArr.forEach( e => selectDate.push(e.date));
      if(selectDate.length>0){
        let params = {examDayList:selectDate};
        postAction("/exam/bizExamDay/batchAdd",params).then((res)=>{
          if(res.success){
            alert(res.message);
            this.getAllDay();
          }
        });
      }
    },
    checkedAll(checkDayObj){
      this.mouths.forEach(moth=>{
        if(moth===checkDayObj.mouth) {
          if(checkDayObj.checkedAll===false){
            this.choseCheckAll = false
          }
        }
      })
    },
    changeYear() {
      this.choseWorkDay = ''
      this.choseCheckAll = false
      this.mouths = []
      for (let i = 1; i <= 12; i++) {
        if (i < 10) {
          this.mouths.push(moment().format('YYYY') + '-' + '0' + i)
        } else {
          this.mouths.push(moment().format('YYYY') + '-' + i)
        }
        
      }
    },
    onClick(day) {
      if (day.select) day.select = false
      else day.select = true
    },
    check(work, noWork) {
      // 选择工作日、非工作日、全选、全不选后，先清空选中项，再添加选中数据
      this.checkedDates = []
      this.mouths.forEach(mouth => {
        this.$refs[mouth][0].checkWork(true, work)
        this.$refs[mouth][0].checkWork(false, noWork)
      })
    },
    // 获取--最终--全部选中的数据
    checkedDate(checkDayObj) {
      console.log(checkDayObj)
      if (this.allCheckDay && this.allCheckDay.length > 0) {
        const mouthObj = this.allCheckDay.findIndex((mouthDay) => mouthDay.mouth === checkDayObj.mouth)
        if (mouthObj !== -1) {
          this.allCheckDay[mouthObj].checkedDay = checkDayObj.checkedDay
        } else this.allCheckDay.push(checkDayObj)
      } else this.allCheckDay.push(checkDayObj)
      this.checkedDates = []
      this.allCheckDay.forEach(mouth => {
        this.checkedDates = [...this.checkedDates, ...mouth.checkedDay]
      })
      /** 
       * 日期列表转化字符串
       * cDatelist为最终list，按需要转化
       * 例：["2019-01-01", "2019-01-02", "2019-01-03", "2019-01-04", "2019-01-05"]
       */
      const checkedDates = [...this.checkedDates]
      const cDatelist = []
      checkedDates.forEach(val => {
        cDatelist.push(val.date)
      })
    },
    changeWorkDay(value){
      this.choseCheckAll = false
      if(value){
        if(value === '工作日'){
          this.check(true,false)
        }else if(value === '非工作日'){
          this.check(false,true)
        }
      }
    },
    changeCheckAll(value){
      this.choseWorkDay = ''
      if(value){
        this.check(true,true)
      }else{
        this.check(false,false)
      }
    }

  }
}
</script>
<style lang="scss" scoped>
.calendar{
  .chose_sd{
    margin: 5px 5%;
    display: flex;
    flex-wrap: wrap;
    align-content: space-between;
  .choseYear {
     width: 80px;margin-top: -2px;
     
    /deep/ .ivu-input{
      height: 25px;
      margin-left: 3px;
    }
   }
    .chose_detail{
      margin-right:5px;
      margin-left: 20px;
    }
  }
  .row{
      display: flex;
      flex-wrap: wrap;
      align-content: space-between;
      justify-content: flex-start;
      margin: 5px  20px ;
  }
}

</style>

