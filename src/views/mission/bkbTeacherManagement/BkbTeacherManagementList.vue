<template>
  <a-card :bordered='false'>
    <!-- 查询区域 -->
    <div class='table-page-search-wrapper'>
      <a-form layout='inline' @keyup.enter.native='searchQuery'>
        <a-row :gutter='24'>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='演讲老师名称'>
              <a-input placeholder='请输入演讲老师名称' v-model='queryParam.speechTeacher'></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='演讲级别'>
              <j-dict-select-tag placeholder='请选择状态' v-model='queryParam.speechLevel' dictCode='speech_level' />
              <!--              <a-input placeholder="请输入演讲级别" v-model="queryParam.speechLevel"></a-input>-->
            </a-form-item>
          </a-col>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <span style='float: left;overflow: hidden;' class='table-page-search-submitButtons'>
              <a-button type='primary' @click='searchQuery' icon='search'>查询</a-button>
              <a-button type='primary' @click='searchReset' icon='reload' style='margin-left: 8px'>重置</a-button>
<!--              <a @click='handleToggleSearch' style='margin-left: 8px'>-->
<!--                {{ toggleSearchStatus ? '收起' : '展开' }}-->
<!--                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />-->
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
<!--      <a-button type='primary' icon='download' @click="handleExportXls('老师管理')">导出</a-button>-->
<!--      <a-upload name='file' :showUploadList='false' :multiple='false' :headers='tokenHeader' :action='importExcelUrl'-->
<!--                @change='handleImportExcel'>-->
<!--        <a-button type='primary' icon='import'>导入</a-button>-->
<!--      </a-upload>-->
<!--      &lt;!&ndash; 高级查询区域 &ndash;&gt;-->
<!--      <j-super-query :fieldList='superFieldList' ref='superQueryModal'-->
<!--                     @handleSuperQuery='handleSuperQuery'></j-super-query>-->
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
<!--      <div class='ant-alert ant-alert-info' style='margin-bottom: 16px;'>-->
<!--        <i class='anticon anticon-info-circle ant-alert-icon'></i> 已选择 <a-->
<!--        style='font-weight: 600'>{{ selectedRowKeys.length }}</a>项-->
<!--        <a style='margin-left: 24px' @click='onClearSelected'>清空</a>-->
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
                                     <a @click='handleDetail(record)'>详情</a>



          <span >
            <a-divider type='vertical' />
 <a @click='handleEdit(record)'>编辑</a>
          </span>
                    <span>
            <a-divider type='vertical' />
                            <a @click='handleDetail2(record)'>预约人数</a>

          </span>
                    <span>
            <a-divider type='vertical' />
                <a-popconfirm title='确定删除吗?' @confirm='() => handleDelete(record.id)'>
                  <a>删除</a>
                </a-popconfirm>
          </span>


        </span>

      </a-table>
    </div>

    <bkb-teacher-management-modal ref='modalForm' @ok='modalFormOk'></bkb-teacher-management-modal>
    <bkb-teacher-management-people-modal ref='modalForm2' @ok='modalFormOk'></bkb-teacher-management-people-modal>

  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import BkbTeacherManagementModal from './modules/BkbTeacherManagementModal'
import { getAction } from '@api/manage'
import BkbTeacherManagementPeopleModal
  from '@views/mission/bkbTeacherManagement/bkb_teacher_management_people/BkbTeacherManagementPeopleModal'

export default {
  name: 'BkbTeacherManagementList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    BkbTeacherManagementModal,
    BkbTeacherManagementPeopleModal
  },
  data() {
    return {
      description: '老师管理管理页面',
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
          title: '头像',
          align: 'center',
          dataIndex: 'headImage',
          scopedSlots: { customRender: 'imgSlot' }
        },
        {
          title: '演讲老师名称',
          align: 'center',
          dataIndex: 'speechTeacher'
        },
        {
          title: '演讲年限',
          align: 'center',
          dataIndex: 'speechYear',
          customRender: function(t, r, index) {
            return t + "年"
          }
        },
        {
          title: '演讲级别',
          align: 'center',
          dataIndex: 'speechLevel_dictText'
        },
        {
          title: '演讲费用',
          align: 'center',
          dataIndex: 'speechExpenses'
        },
        {
          title: '联系电话',
          align: 'center',
          dataIndex: 'phone'
        },
        // {
        //   title: '图片上传',
        //   align: 'center',
        //   dataIndex: 'uploadPictures',
        //   scopedSlots: { customRender: 'imgSlot' }
        // },
        // {
        //   title: '个人简介',
        //   align: 'center',
        //   dataIndex: 'personalProfile'
        // },
        {
          title: '预约订单',
          align: 'center',
          dataIndex: 'bookingOrder'
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
        list2: '/mission/bkbTeacherManagementPeople/list',
        list: '/mission/bkbTeacherManagement/list',
        delete: '/mission/bkbTeacherManagement/delete',
        deleteBatch: '/mission/bkbTeacherManagement/deleteBatch',
        exportXlsUrl: '/mission/bkbTeacherManagement/exportXls',
        importExcelUrl: 'mission/bkbTeacherManagement/importExcel'

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
    handleDetail2: function(record) {
      getAction(this.url.list2, { teacherId: record.id }).then(res => {
        if (res.success) {
          this.$refs.modalForm2.edit(res.result.records)
          this.$refs.modalForm2.title = '老师预约人数'
          this.$refs.modalForm2.mode = record
          this.$refs.modalForm2.queryParam = {}
          this.$refs.modalForm2.disableSubmit = true
        }
      })
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'headImage', text: '头像', dictCode: '' })
      fieldList.push({ type: 'string', value: 'speechTeacher', text: '演讲老师名称', dictCode: '' })
      fieldList.push({ type: 'int', value: 'speechYear', text: '演讲年限', dictCode: '' })
      fieldList.push({ type: 'int', value: 'speechLevel', text: '演讲级别', dictCode: 'speech_level' })
      fieldList.push({ type: 'string', value: 'speechExpenses', text: '演讲费用', dictCode: '' })
      fieldList.push({ type: 'string', value: 'phone', text: '联系电话', dictCode: '' })
      fieldList.push({ type: 'string', value: 'uploadPictures', text: '图片上传', dictCode: '' })
      fieldList.push({ type: 'string', value: 'personalProfile', text: '个人简介', dictCode: '' })
      fieldList.push({ type: 'int', value: 'bookingOrder', text: '预约订单', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>