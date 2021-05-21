<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label="所属工种" :labelCol="labelCol" :wrapperCol="wrapperCol">
  	          <!-- <j-tree-select
                ref="treeSelect"
                placeholder="请选择所属工种id"
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
            <a-form-item label="题型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['subType']" :trigger-change="true" dictCode="subject_type" placeholder="请选择题型" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="题干" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['subDesc']" placeholder="请输入题干"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="选择题答案" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['switchResult']" placeholder="请输入选择题答案（a,b,c）"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="判断题答案" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['judgeResult']" placeholder="请输入判断题答案(y,n)"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="a选项" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['resultA']" placeholder="请输入a选项"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col  :sm="24" >
            <a-form-item label="a选项图片" hasFeedback :labelCol="labelCol" :wrapperCol="wrapperCol">
              <CropperImage  v-model="model.picA" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="b选项" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['resultB']" placeholder="请输入b选项"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col  :sm="24" >
            <a-form-item label="b选项图片" hasFeedback :labelCol="labelCol" :wrapperCol="wrapperCol">
              <CropperImage  v-model="model.picB" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="c选项" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['resultC']" placeholder="请输入c选项"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col  :sm="24" >
            <a-form-item label="c选项图片" hasFeedback :labelCol="labelCol" :wrapperCol="wrapperCol">
              <CropperImage  v-model="model.picC" />
            </a-form-item>
          </a-col>
          <!-- <a-col :span="24">
            <a-form-item label="正确选项" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['resultYes']" placeholder="请输入正确选项"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="错误选项" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['resultNo']" placeholder="请输入错误选项"  ></a-input>
            </a-form-item>
          </a-col> -->
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
  import CropperImage from '@comp/dfzt/CropperImage'
  export default {
    name: 'BizSubjectForm',
    components: {CropperImage},
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
        picA:'',
        picB:'',
        picC:'',
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
          add: "/exam/bizSubject/add",
          edit: "/exam/bizSubject/edit",
          queryById: "/exam/bizSubject/queryById"
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
      changePic(val) {
        let arr = val.split('?')
        let pic = arr[0]
        this.picA = pic
      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'workTypeId','subType','subDesc','switchResult','judgeResult','resultA','resultB','resultC','picA','picB','picC','resultYes','resultNo'))
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
        this.form.setFieldsValue(pick(row,'workTypeId','subType','subDesc','switchResult','judgeResult','resultA','resultB','resultC','picA','picB','picC','resultYes','resultNo'))
      },
    }
  }
</script>