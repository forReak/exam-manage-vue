<template>
  <a-spin :spinning="confirmLoading">
    <!-- <img :src="data1" alt="" style="display:block;margin:0 auto;margin-bottom:50px;width:10vw"> -->

          
<!-- v-decorator="['userPic']" -->
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <!-- <a-col :span="24">
            <a-form-item label="培训机构" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['createBy']" placeholder="请输入培训机构"></a-input>
            </a-form-item>
          </a-col> -->
          <a-col  :sm="24" >
            <a-form-item label="考生照片" hasFeedback :labelCol="labelCol" :wrapperCol="wrapperCol">
              <CropperImage  v-model="model.userPic"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="姓名" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['userName']" placeholder="请输入姓名"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="性别" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <!-- <a-input v-decorator="['sex']" placeholder="请输入性别"></a-input> -->
              <!-- <j-dict-select-tag
                placeholder="请输入性别"
                dictCode="user_sex"
                 v-decorator="['sex']" 
              /> -->
            <a-select v-decorator="['sex']" placeholder="请选择性别">
              <a-select-option :value="'男'">男</a-select-option>
              <a-select-option :value="'女'">女</a-select-option>
            </a-select>

            </a-form-item>
          </a-col>
          
          <a-col :span="24">
            <a-form-item label="身份证号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['idCard']" placeholder="请输入身份证号"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="联系电话" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['phone']" placeholder="请输入联系电话"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="工种" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <!-- <j-tree-select
                ref="treeSelect"
                placeholder="请选择工种"
                v-decorator="['workTypeId']"
                dict="biz_work_type,work_type_name,id"
                pidValue="0"
                >
              </j-tree-select>-->
              <j-tree-select
                v-decorator="['workTypeId']"
                placeholder="请选择所属工种"
                dict="biz_work_type,work_type_name,id"
                pidField="parent_id"
                hasChildField="has_child"
                pidValue="0"
              ></j-tree-select>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="考试时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date
                placeholder="请选择考试时间"
                v-decorator="['examTime']"
                :trigger-change="true"
                date-format="YYYY-MM-DD"
                style="width: 100%"
              />
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
import CropperImage from '@comp/dfzt/CropperImage'

export default {
  name: 'BizSignListForm',
  components: {CropperImage},
  props: {
    //流程表单data
    formData: {
      type: Object,
      default: () => {},
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

  created(){
      this.userPic = "";
  },
  data() {
    return {
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 },
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 },
      },
      userPic:"",
      data1: '',
      form: this.$form.createForm(this),
      model: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      confirmLoading: false,
      validatorRules: {},
      url: {
        add: '/exam/bizSignList/add',
        edit: '/exam/bizSignList/edit',
        queryById: '/exam/bizSignList/queryById'
      }
    }
  },
  computed: {
    formDisabled() {
      if (this.formBpm === true) {
        if (this.formData.disabled === false) {
          return false
        }
        return true
      }
      return this.disabled
    },
    showFlowSubmitButton() {
      if (this.formBpm === true) {
        if (this.formData.disabled === false) {
          return true
        }
      }
      return false
    }
  },
  created() {
    //如果是流程中表单，则需要加载流程表单data
    this.showFlowData()
  },
  methods: {
    doImgUpload(){
      alert();
    },
    changePic(val) {
      console.log(val)
      let arr = val.split('?')
      let pic = arr[0]
      this.userPic = pic
    },
    add() {
      this.edit({})
    },
    edit(record) {
      this.data1 = "data:image/png;base64,"+record.userPic;
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(
          pick(
            this.model,
            'createBy',
            'userName',
            'sex',
            'idCard',
            'phone',
            'workTypeId',
            'examTime',
            'createTime',
            'userPic'
          )
        )
      })
    },
    //渲染流程表单数据
    showFlowData() {
      if (this.formBpm === true) {
        let params = { id: this.formData.dataId }
        getAction(this.url.queryById, params).then(res => {
          if (res.success) {
            this.edit(res.result)
          }
        })
      }
    },
    submitForm() {
      const that = this
      // 触发表单验证
      this.form.validateFields((err, values) => {
        if (!err) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }
          let formData = Object.assign(this.model, values)
          console.log('表单提交数据', formData)
          httpAction(httpurl, formData, method)
            .then(res => {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
              } else {
                that.$message.warning(res.message)
              }
            })
            .finally(() => {
              that.confirmLoading = false
            })
        }
      })
    },
    popupCallback(row) {
      this.form.setFieldsValue(
        pick(row, 'createBy', 'userName', 'sex', 'idCard', 'phone', 'workTypeId', 'examTime', 'createTime', 'userPic')
      )
    }
  }
}
</script>