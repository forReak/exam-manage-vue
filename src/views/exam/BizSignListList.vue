<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="培训机构">
              <a-input placeholder="请输入培训机构" v-model="queryParam.createBy"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="姓名">
              <a-input placeholder="请输入姓名" v-model="queryParam.userName"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="身份证号">
                <a-input placeholder="请输入身份证号" v-model="queryParam.idCard"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="工种">
                <!-- <a-input placeholder="请输入工种" v-model="queryParam.workTypeId"></a-input> -->
                <j-tree-select
                v-model="queryParam.workTypeId"
                placeholder="请选择所属工种"
                dict="biz_work_type,work_type_name,id"
                pidField="parent_id"
                hasChildField="has_child"
                pidValue="0"
                >
                </j-tree-select>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="考试时间">
                <j-date style="width:100%" date-format="YYYY-MM-DD" placeholder="请选择考试时间" v-model="queryParam.examTime"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="12" :lg="11" :md="12" :sm="24">
              <a-form-item label="报名时间">
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择开始时间" class="query-group-cust" v-model="queryParam.createTime_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择结束时间" class="query-group-cust" v-model="queryParam.createTime_end"></j-date>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">手动新增</a-button>
      <!-- <a-button type="primary" icon="download" @click="handleExportXls('报名信息')">导出</a-button> -->
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">上传报名文件</a-button>
      </a-upload>
      <!-- <a-button @click="customImport" type="primary" icon="plus">批量报名</a-button> -->
      <!-- 高级查询区域 -->
      <!-- <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query> -->
      <!-- <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown> -->
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <biz-sign-list-modal ref="modalForm" @ok="modalFormOk"></biz-sign-list-modal>
    <signImport ref="signImport" ></signImport>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import BizSignListModal from './modules/BizSignListModal'
  import signImport from './modules/SigncustomImport'

  export default {
    name: 'BizSignListList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      BizSignListModal,signImport
    },
    data () {
      return {
        description: '报名信息管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'培训机构',
            align:"center",
            dataIndex: 'createBy_dictText'
          },
          {
            title:'姓名',
            align:"center",
            dataIndex: 'userName'
          },
          {
            title:'性别',
            align:"center",
            dataIndex: 'sex'
          },
          {
            title:'身份证号',
            align:"center",
            dataIndex: 'idCard'
          },
          {
            title:'联系电话',
            align:"center",
            dataIndex: 'phone'
          },
          {
            title:'工种',
            align:"center",
            dataIndex: 'workTypeId_dictText'
          },
          {
            title:'考试时间',
            align:"center",
            dataIndex: 'examTime'
          },
          {
            title:'报名时间',
            align:"center",
            dataIndex: 'createTime'
          },
          {
            title:'是否参加考试',
            align:"center",
            dataIndex: 'firstJoinTime',
            customRender: function (text) {
              if(text !== null){
                return "是"
              }else{
                return ""
              }
            },
          },
          {
            title:'是否交卷',
            align:"center",
            dataIndex: 'hasExam',
            customRender: function (text) {
              if(text == 0){
                return ""
              }else{
                return "是"
              }
            },
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/exam/bizSignList/list",
          delete: "/exam/bizSignList/delete",
          deleteBatch: "/exam/bizSignList/deleteBatch",
          exportXlsUrl: "/exam/bizSignList/exportXls",
          importExcelUrl: "exam/bizSignList/customImportExcel",
          // importExcelUrl: "online/cgform/api/importXls/890b01d025ba4ac1926d3fc7eb3ea569",
          
        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
    this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      customImport(){
          this.$refs.signImport.visible = true;
      },
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'createBy',text:'培训机构',dictCode:''})
        fieldList.push({type:'string',value:'userName',text:'姓名',dictCode:''})
        fieldList.push({type:'string',value:'sex',text:'性别',dictCode:''})
        fieldList.push({type:'string',value:'idCard',text:'身份证号',dictCode:''})
        fieldList.push({type:'string',value:'phone',text:'联系电话',dictCode:''})
        fieldList.push({type:'string',value:'workTypeId',text:'工种'})
        fieldList.push({type:'datetime',value:'examTime',text:'考试时间'})
        fieldList.push({type:'datetime',value:'createTime',text:'报名时间'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>