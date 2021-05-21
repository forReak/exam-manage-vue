<template>
   <div>
    <index-chart v-if="indexStyle==1"></index-chart>
    <index-bdc v-if="indexStyle==2"></index-bdc>
    <index-task v-if="indexStyle==3"></index-task>
    <div style="width: 100%;text-align: right;margin-top: 20px;display:none">
      请选择首页样式：
      <a-radio-group v-model="indexStyle">
        <a-radio :value="1">统计图表</a-radio>
        <a-radio :value="2">统计图表2</a-radio>
        <a-radio :value="3">任务表格</a-radio>
      </a-radio-group>
    </div>
  </div>
</template>

<script>
  import IndexChart from './IndexChart'
  import IndexTask from "./IndexTask"
  import IndexBdc from './IndexBdc'

  export default {
    name: "Analysis",
    components: {
      IndexChart,
      IndexTask,
      IndexBdc
    },
    data() {
      return {
        indexStyle:1
      }
    },
    created() {

    },
    methods: {

    }
  }
</script>

<style>
.borderDiv{
  width:70vw;
  height:30vh;
  /* border: 1px solid black; */
  margin: 0 auto;
  margin-top:5vh;
}
.title{
  font-size: 20px;
  font-weight: bolder
}
.box{
  width: 14vw;
  height: 20vh;
  border: 1px solid slategray;
  display: inline-block;
  margin-left: 2vw;
  margin-top: 2vh;
}
</style>