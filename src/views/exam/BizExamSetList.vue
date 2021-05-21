<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="所属工种">
              <!-- <a-input placeholder="请输入所属工种id" v-model="queryParam.workTypeId"></a-input> -->
              <j-tree-select
                v-model="queryParam.workTypeId"
                placeholder="请选择所属工种"
                dict="biz_work_type,work_type_name,id"
                pidField="parent_id"
                hasChildField="has_child"
                pidValue="0"
                >
                </j-tree-select>
              {{ workTypeId }}
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button
                type="primary"
                @click="searchReset"
                icon="reload"
                style="margin-left: 8px"
              >重置</a-button>
              <!-- <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" /> -->
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <!-- <a-button type="primary" icon="download" @click="handleExportXls('工种考试设置')">导出</a-button>
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload> -->
      <!-- 高级查询区域 -->
      <!-- <j-super-query
        :fieldList="superFieldList"
        ref="superQueryModal"
        @handleSuperQuery="handleSuperQuery"
      ></j-super-query> -->
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete" />删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px">
          批量操作
          <a-icon type="down" />
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
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
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img
            v-else
            :src="getImgView(text)"
            height="25px"
            alt
            style="max-width:80px;font-size: 12px;font-style: italic;"
          />
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)"
          >下载</a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">
              更多
              <a-icon type="down" />
            </a>
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

    <biz-exam-set-modal ref="modalForm" @ok="modalFormOk"></biz-exam-set-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import BizExamSetModal from './modules/BizExamSetModal'
import JTreeSelect from '@/components/jeecg/JTreeSelect'

export default {
  name: 'BizExamSetList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    BizExamSetModal,
    JTreeSelect
  },
  data() {
    return {
      workTypeId:"",
      description: '工种考试设置管理页面',
      // 表头
      columns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        // {
        //   title: '所属工种',
        //   align: 'center',
        //   dataIndex: 'workTypeId'
        // },
        {
          title: '所属工种',
          align: 'center',
          dataIndex: 'workTypeId_dictText'
        },
        {
          title: '考试时长(分钟)',
          align: 'center',
          dataIndex: 'time'
        },
        {
          title: '试卷总分',
          align: 'center',
          dataIndex: 'totalScore'
        },
        {
          title: '分数线',
          align: 'center',
          dataIndex: 'passScore'
        },
        {
          title: '选择题分值',
          align: 'center',
          dataIndex: 'switchScore'
        },
        {
          title: '选择题数量',
          align: 'center',
          dataIndex: 'switchNum'
        },
        {
          title: '判断题分值',
          align: 'center',
          dataIndex: 'judgeScore'
        },
        {
          title: '判断题数量',
          align: 'center',
          dataIndex: 'judgeNum'
        },
        {
          title: '开始考试时间',
          align: 'center',
          dataIndex: 'beginExamTime'
        },
        {
          title: '结束考试时间',
          align: 'center',
          dataIndex: 'endExamTime'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          fixed: 'right',
          width: 147,
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/exam/bizExamSet/list',
        delete: '/exam/bizExamSet/delete',
        deleteBatch: '/exam/bizExamSet/deleteBatch',
        exportXlsUrl: '/exam/bizExamSet/exportXls',
        importExcelUrl: 'exam/bizExamSet/importExcel'
      },
      dictOptions: {},
      superFieldList: []
    }
  },
  created() {
    this.getSuperFieldList()
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'workTypeId', text: '所属工种id' })
      fieldList.push({ type: 'int', value: 'time', text: '考试时长(分钟)', dictCode: '' })
      fieldList.push({ type: 'int', value: 'totalScore', text: '试卷总分', dictCode: '' })
      fieldList.push({ type: 'int', value: 'passScore', text: '分数线', dictCode: '' })
      fieldList.push({ type: 'int', value: 'switchScore', text: '选择题分值', dictCode: '' })
      fieldList.push({ type: 'int', value: 'switchNum', text: '选择题数量', dictCode: '' })
      fieldList.push({ type: 'int', value: 'judgeScore', text: '判断题分值', dictCode: '' })
      fieldList.push({ type: 'int', value: 'judgeNum', text: '判断题数量', dictCode: '' })
      fieldList.push({ type: 'string', value: 'beginExamTime', text: '开始考试时间 HH:MM:ss', dictCode: '' })
      fieldList.push({ type: 'string', value: 'endExamTime', text: '结束考试时间', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>