<template>
  <a-drawer
    placement="right"
    :closable="false"
    width="70%"
    :visible="visible"
    :after-visible-change="afterVisibleChange"
    @close="onClose"
  >
    <span style="font-size: 20pt"></span>
    <span style="font-size: 20pt"></span>
  
      <a-row style="font-size: 20pt" :gutter="24">
        <a-col :span="24">姓名：{{model.userName}}</a-col>
      </a-row>
      <a-row style="font-size: 16pt;margin-top:20px" :gutter="24">
        <a-col :span="24">身份证：{{model.idCard}}</a-col>
      </a-row>
      <a-row style="font-size: 16pt;margin-top:20px" :gutter="24">
        <a-col :span="24">考试科目：{{model.workType_dictText}}</a-col>
      </a-row>
    <a-row style="font-size: 16pt;margin-top:20px" :gutter="24">
        <a-col :span="24">考试时间：{{model.createTime}}</a-col>
      </a-row>
      <a-row style="font-size: 16pt;margin-top:20px" :gutter="24">
        <a-col :span="24">分数：{{model.score}}</a-col>
      </a-row>

      <a-row style="font-size: 16pt;margin-top:20px" :gutter="24">
        <a-col v-if="model.result ==='不通过'" style="color:red" :span="24">结果：{{model.result}} </a-col>
        <a-col v-if="model.result ==='通过'" style="color:green" :span="24">结果：{{model.result}} </a-col>
      </a-row>
      <a-row style="font-size: 14pt;margin-top:20px" :gutter="24">
        <a-col :span="24">判读题：</a-col>
      </a-row>
      <a-card style="margin-bottom: 24px;margin-top: 20px;opacity: 1; border-radius: 8px; background: #ffffff">

        <table>
          <tr>
            <td>序号</td>
            <td style="width:80%">题目</td>
            <td>考生答案</td>
            <td>正确答案</td>
          </tr>
          <tr v-for="(item,i) in userJudgeList" :key="i">
            <td v-if="item.userAnswer !== item.answer" style="color:red">第{{item.i}} 题</td>
            <td v-else >第{{item.i}} 题</td>
            <td>{{item.desc}}</td>
            <td v-if="item.userAnswer === 'y'">正确</td>
            <td v-if="item.userAnswer === 'n'">错误</td>
            <td v-if="item.userAnswer !=='y' && item.userAnswer !=='n'"> 未作答 </td>
            <td v-if="item.answer === 'n'">错误</td>
            <td v-if="item.answer === 'y'">正确</td>
          </tr>
        </table>
      </a-card>

      <a-row style="font-size: 14pt;margin-top:20px" :gutter="24">
        <a-col :span="24">选择题</a-col>
      </a-row>
      <a-card style="margin-bottom: 24px;margin-top: 20px;opacity: 1; border-radius: 8px; background: #ffffff">

        <table>
          <tr>
            <td>序号</td>
            <td style="width:50%">题目</td>
            <td>a选项</td>
            <td>b选项</td>
            <td>c选项</td>
            <td>考生答案</td>
            <td>正确答案</td>
          </tr>
          <tr v-for="(item,i) in userSwitchList" :key="i">
            <td v-if="toLower(item.userAnswer,item.answer)" style="color:red">第{{item.i}} 题</td>
            <td v-else> 第{{item.i}} 题</td>
            <td>{{item.desc}}</td>
            <td>{{item.a}}</td>
            <td>{{item.b}}</td>
            <td>{{item.c}}</td>
            <td>{{item.userAnswer}} </td>
            <td>{{item.answer}}</td>
          </tr>
        </table>
      </a-card>
  </a-drawer>
</template>

<script>

//设备配置中的详情页
import { postAction } from '@api/manage'

export default {
  name: "subjectDetail",
  components: {

  },
  data(){
    return {
      visible:false,
      model:{},
      userJudgeList:[],
      userSwitchList:[],
      judgeAnswer:[],
      switchAnswer:[],
      url: {
        querySubjct: '/exam/bizSubject/getByIdList',
      },

    }
  },
  created () {

  },
  methods: {
    detail(record) {
      console.log(record);
      this.initRecordInfo(record);
    },

    initRecordInfo(record){
      this.model = record;
      this.judgeAnswer = JSON.parse(record.judgeAnswer);
      this.switchAnswer = JSON.parse(record.switchAnswer);
      this.getSubjectInfo();
    },

    getSubjectInfo(){
      let judgeData = {ids:JSON.parse(this.model.judgeList)};
      postAction(this.url.querySubjct,judgeData).then((res) => {
        if (res.success) {
          let judgeList = res.result;
          let judgeAnswer = this.judgeAnswer;
          for(var x = 0;x<judgeAnswer.length;x++){
            for(var y = 0 ; y < judgeList.length;y++){
              if(judgeAnswer[x].subjectId === judgeList[y].id){
                this.userJudgeList.push({
                  userAnswer:judgeAnswer[x].answer,
                  answer:judgeList[y].judgeResult,
                  desc:judgeList[y].subDesc,
                  i:(judgeAnswer[x].index + 1)
                  });
                break;
              }
            }
          }
          console.log("判断题")
          console.log(this.userJudgeList)
          
        } else {
           this.$message.warning(res.message)
        }
      });

      let switchData = {ids:JSON.parse(this.model.switchList)};
      postAction(this.url.querySubjct,switchData).then((res) => {
        if (res.success) {
          let switchList = res.result;
          let switchAnswer = this.switchAnswer;
          
          for(var x = 0;x<switchAnswer.length;x++){
            for(var y = 0 ; y < switchList.length;y++){
              if(switchAnswer[x].subjectId === switchList[y].id){
                this.userSwitchList.push({
                  userAnswer:switchAnswer[x].answer,
                  answer:switchList[y].switchResult,
                  desc:switchList[y].subDesc,
                  a:switchList[y].resultA,
                  b:switchList[y].resultB,
                  c:switchList[y].resultC,
                  i:(switchAnswer[x].index + 1)
                  });
                break;
              }
            }
          }
          console.log("选择题")
          console.log(this.userSwitchList)
          
        } else {
           this.$message.warning(res.message)
        }
      });

    },
    toLower( userAnswer,answer){
      let a = '';
      if(answer === 'A'){
        a = 'a';
      }else if (answer === 'B'){
        a = 'b';
      }else{
        a = 'c';
      }
      if(userAnswer !== a){
        return true;
      }else{
        return false;
      }
    },

    onClose() {
      // this.model = {}
      // this.$emit('onClose')
      this.visible = false
    },

    afterVisibleChange(val) {
      console.log('visible', val)
    },
  }
}
</script>

<style scoped>
  table {
    width: 100%;
    border-collapse: collapse;
  }
  th,
  td {
    border: 1px solid #eee;
    text-align: center;
    padding: 10px 0;
    background: #fafafa;
  }
  td {
    background: none;
  }
</style>