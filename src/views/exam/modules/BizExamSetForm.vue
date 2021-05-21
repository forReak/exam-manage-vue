<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label="所属工种" :labelCol="labelCol" :wrapperCol="wrapperCol">
  	          <!-- <j-tree-select
                ref="treeSelect"
                placeholder="请选择所属工种"
                v-decorator="['workTypeId']"
                dict="biz_work_type,work_type_name,id"
                pidValue="0"
                >
              </j-tree-select> -->
              <j-tree-select
                v-decorator="['workTypeId']"
                placeholder="请选择所属工种"
                dict="biz_work_type,work_type_name,id"
                pidField="parent_id"
                hasChildField="has_child"
                pidValue="0"
                >
                </j-tree-select>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="考试时长(分钟)" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['time']" placeholder="请输入考试时长(分钟)" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="试卷总分" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['totalScore']" placeholder="请输入试卷总分" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="分数线" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['passScore']" placeholder="请输入分数线" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="选择题分值" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['switchScore']" placeholder="请输入选择题分值" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="选择题数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['switchNum']" placeholder="请输入选择题数量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="判断题分值" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['judgeScore']" placeholder="请输入判断题分值" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="判断题数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['judgeNum']" placeholder="请输入判断题数量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="开始考试时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['beginExamTime']" placeholder="请输入开始考试时间"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="结束考试时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['endExamTime']" placeholder="请输入结束考试时间"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col v-if="showFlowSubmitButton" :span="24" style="text-align: center">
            <a-button @click="submitForm">提 交</a-button>
          </a-col>
        </a-row>
      </a-form>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: 'BizExamSetForm',
    components: {
    },
    props: {
      //流程表单data
      formData: {
        type: Object,
        default: ()=>{},
        required: false
      },
      //表单模式：true流程表单 false普通表单
      formBpm: {
        type: Boolean,
        default: false,
        required: false
      },
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
        form: this.$form.createForm(this),
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
        },
        url: {
          add: "/exam/bizExamSet/add",
          edit: "/exam/bizExamSet/edit",
          queryById: "/exam/bizExamSet/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return false
          }
          return true
        }
        return this.disabled
      },
      showFlowSubmitButton(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return true
          }
        }
        return false
      }
    },
    created () {
      //如果是流程中表单，则需要加载流程表单data
      this.showFlowData();
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        //todo 禁用更改工种
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'workTypeId','time','totalScore','passScore','switchScore','switchNum','judgeScore','judgeNum','beginExamTime','endExamTime'))
        })
      },
      //渲染流程表单数据
      showFlowData(){
        if(this.formBpm === true){
          let params = {id:this.formData.dataId};
          getAction(this.url.queryById,params).then((res)=>{
            if(res.success){
              this.edit (res.result);
            }
          });
        }
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }
         
        })
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'workTypeId','time','totalScore','passScore','switchScore','switchNum','judgeScore','judgeNum','beginExamTime','endExamTime'))
      },
    }
  }
</script>