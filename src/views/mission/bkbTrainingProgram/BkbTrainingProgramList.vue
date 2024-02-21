<template>
  <a-card :bordered='false'>
    <!-- 查询区域 -->
    <div class='table-page-search-wrapper'>
      <a-form layout='inline' @keyup.enter.native='searchQuery'>
        <a-row :gutter='24'>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='培训标题'>
              <a-input placeholder='请输入培训标题' v-model='queryParam.title'></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='培训演讲人'>
              <!--              <a-input placeholder='请输入培训演讲人' v-model='queryParam.speakerId'></a-input>-->
              <j-dict-select-tag placeholder='培训演讲人' v-model='queryParam.speakerId'
                                 dictCode='bkb_training_speakers,speaker,id' />
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='状态'>
              <j-dict-select-tag placeholder='请输入状态' v-model='queryParam.status' dictCode='training_status' />
              <!--                <a-input placeholder="请输入活动状态" v-model="queryParam.status"></a-input>-->
            </a-form-item>
          </a-col>
          <template v-if='toggleSearchStatus'>
            <!--            <a-col :xl="6" :lg="7" :md="8" :sm="24">-->
            <!--              <a-form-item label="状态">-->
            <!--                <a-input placeholder="请输入状态" v-model="queryParam.status"></a-input>-->
            <!--              </a-form-item>-->
            <!--            </a-col>-->

          </template>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <span style='float: left;overflow: hidden;' class='table-page-search-submitButtons'>
              <a-button type='primary' @click='searchQuery' icon='search'>查询</a-button>
              <a-button type='primary' @click='searchReset' icon='reload' style='margin-left: 8px'>重置</a-button>
              <!--              <a @click="handleToggleSearch" style="margin-left: 8px">-->
              <!--                {{ toggleSearchStatus ? '收起' : '展开' }}-->
              <!--                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>-->
              <!--              </a>-->
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class='table-operator'>
      <a-button @click='handleAdd' type='primary' icon='plus'>新增</a-button>
      <!--      <a-button type="primary" icon="download" @click="handleExportXls('培训计划')">导出</a-button>-->
      <!--      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">-->
      <!--        <a-button type="primary" icon="import">导入</a-button>-->
      <!--      </a-upload>-->
      <!--      &lt;!&ndash; 高级查询区域 &ndash;&gt;-->
      <!--      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>-->
      <a-dropdown v-if='selectedRowKeys.length > 0'>
        <a-menu slot='overlay'>
          <a-menu-item key='1' @click='batchDel'>
            <a-icon type='delete' />
            删除
          </a-menu-item>
        </a-menu>
        <a-button style='margin-left: 8px'> 批量操作
          <a-icon type='down' />
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <!--      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">-->
      <!--        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项-->
      <!--        <a style="margin-left: 24px" @click="onClearSelected">清空</a>-->
      <!--      </div>-->

      <a-table
        ref='table'
        size='middle'
        :scroll='{x:true}'
        bordered
        rowKey='id'
        :columns='columns'
        :dataSource='dataSource'
        :pagination='ipagination'
        :loading='loading'
        class='j-table-force-nowrap'
        @change='handleTableChange'>

        <template slot='htmlSlot' slot-scope='text'>
          <div v-html='text'></div>
        </template>
        <template slot='imgSlot' slot-scope='text,record'>
          <span v-if='!text' style='font-size: 12px;font-style: italic;'>无图片</span>
          <img v-else :src='getImgView(text)' :preview='record.id' height='25px' alt=''
               style='max-width:80px;font-size: 12px;font-style: italic;' />
        </template>
        <template slot='fileSlot' slot-scope='text'>
          <span v-if='!text' style='font-size: 12px;font-style: italic;'>无文件</span>
          <a-button
            v-else
            :ghost='true'
            type='primary'
            icon='download'
            size='small'
            @click='downloadFile(text)'>
            下载
          </a-button>
        </template>

        <span slot='action' slot-scope='text, record'>
          <span v-if='record.status!==3 && record.status!==2'>

          <a @click='handleEdit(record)'>编辑</a>
                       <a-divider type='vertical' />

          </span>
          <span v-if='record.status===3'>
            <a @click='handleEdit2(record)'>培训结果查看</a>
                                   <a-divider type='vertical' />

          </span>

          <span>

            <a @click='handleDetail(record)'>详情</a>
                       <a-divider v-if='record.status===0 ||record.status===3' type='vertical' />
          </span>
          <span v-if='record.status===0 ||record.status===3'>
                          <a-popconfirm title='确定删除吗?' @confirm='() => handleDelete(record.id)'>
                  <a>删除</a>
                </a-popconfirm>
          </span>
        </span>

      </a-table>
    </div>

    <bkb-training-program-modal ref='modalForm' @ok='modalFormOk'></bkb-training-program-modal>
    <bkb-training-program-2-modal ref='modalForm2' @ok='modalFormOk'></bkb-training-program-2-modal>
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import BkbTrainingProgramModal from './modules/BkbTrainingProgramModal'
import { filterDictTextByCache, filterMultiDictText } from '@comp/dict/JDictSelectUtil'
import { loadCategoryData } from '@api/api'
import BkbTrainingProgram2Modal from '@views/mission/bkbTrainingProgram/modules/BkbTrainingProgram2Modal'

export default {
  name: 'BkbTrainingProgramList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    BkbTrainingProgramModal,
    BkbTrainingProgram2Modal
  },
  data() {
    return {
      description: '培训计划管理页面',
      // 表头
      columns: [
        {
          title: '序号',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '培训标题',
          align: 'center',
          dataIndex: 'title'
        },
        {
          title: '培训类型',
          align: 'center',
          dataIndex: 'type',
          customRender: (text) => {
            //字典值替换通用方法
            return filterDictTextByCache('train_type', text)
          }
        },
        {
          title: '培训地址',
          align: 'center',
          dataIndex: 'addr'
        },
        {
          title: '开始时间',
          align: 'center',
          dataIndex: 'startDate'
        },
        {
          title: '培训时长',
          align: 'center',
          dataIndex: 'duration',
          customRender: function(t, r, index) {
            return t + '小时'
          }
        },
        {
          title: '获得积分',
          align: 'center',
          dataIndex: 'integral'
        },
        {
          title: '培训演讲人',
          align: 'center',
          dataIndex: 'speakerId_dictText'
        },
        {
          title: '状态',
          align: 'center',
          dataIndex: 'status_dictText'
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
        list: '/mission/bkbTrainingProgram/list',
        delete: '/mission/bkbTrainingProgram/delete',
        deleteBatch: '/mission/bkbTrainingProgram/deleteBatch',
        exportXlsUrl: '/mission/bkbTrainingProgram/exportXls',
        importExcelUrl: 'mission/bkbTrainingProgram/importExcel'

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
    initDictConfig() {

    },
    handleEdit2: function(record) {
      this.$refs.modalForm2.title = '查看'
      this.$refs.modalForm2.edit(record)
      this.$refs.modalForm2.disableSubmit = false
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'title', text: '培训标题', dictCode: '' })
      fieldList.push({ type: 'string', value: 'coverUrl', text: '封面', dictCode: '' })
      fieldList.push({ type: 'int', value: 'type', text: '培训类型' })
      fieldList.push({ type: 'string', value: 'addr', text: '培训地址', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'startDate', text: '开始时间' })
      fieldList.push({ type: 'int', value: 'duration', text: '培训时长', dictCode: '' })
      fieldList.push({ type: 'int', value: 'integral', text: '获得积分', dictCode: '' })
      fieldList.push({
        type: 'string',
        value: 'speakerId',
        text: '培训演讲人',
        dictCode: '`bkb_training_speakers,speaker,id,`'
      })
      fieldList.push({ type: 'int', value: 'status', text: '状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'content', text: '培训内容简介', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>