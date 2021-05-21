<template>
  <a-modal
    :title="title"
    :width="850"
    :visible="visible"
    @ok="handleOk"
    @cancel="close"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form>
        <a-row :gutter="24">
          <a-col :md="12" :sm="8">
            <!-- <a-form-item label="项目名称">
              <a-select v-model="projectId" placeholder="请选择项目" mode="default" style="width: 100%">
                <a-select-option
                  :value="item.id"
                  v-for="(item, index) in projectList"
                  :key="index"
                >{{ item.name }}</a-select-option>
              </a-select>
            </a-form-item> -->
            <a-form-item label="第一步：请选择本次报名工种">
                <!-- <a-input placeholder="请输入工种" v-model="queryParam.workTypeId"></a-input> -->
                <j-tree-select
                v-model="workTypeId"
                placeholder="请选择所属工种"
                dict="biz_work_type,work_type_name,id"
                pidField="parent_id"
                hasChildField="has_child"
                pidValue="0"
                >
                </j-tree-select>
              </a-form-item>
          </a-col>
        </a-row>
        <span>第二步：请上传报名文件</span>
        <br><br>
        <!--  此处省略部分代码...... -->
        <a-button @click="handleImportXls" type="primary" icon="upload">上传Excel文件</a-button>
        <!--  此处省略部分代码...... -->
        <j-import-modal ref="importModal" :url="getImportUrl()" @ok="importOk"></j-import-modal>
        <!--  此处省略部分代码...... -->
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
import { getAction, postAction } from '@api/manage'
import JCodeEditor from '@/components/jeecg/JCodeEditor'

export default {
  components: {JCodeEditor},

  name: 'signImport',
  
  data() {
    return {
      title: '考试报名',
      visible: false,
      confirmLoading: false,
      workTypeId:'',
      url: {
        add: '/chargeConfig/saveChargeModel',
        queryProject: '/project/projectArchives/getProjectByUsername'
      }
    }
  },

  created() {
    
  },

  methods: {
    
  
    close() {
      this.visible = false
    },

     handleImportXls(){
        this.$refs.importModal.show()
      },
      
      getImportUrl(){
         return '/exam/bizSignList/customImportExcel?workTypeId='+this.workTypeId
        
      },
      importOk(){
        alert('导入成功');
        this.close();
      },
      handleOk() {
        this.close();
      }
  }
}

</script>

<style lang="less" scoped>

</style>