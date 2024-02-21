<template>
  <a-card :bordered='false'>
    <!-- 查询区域 -->
    <div class='table-page-search-wrapper'>
      <a-form layout='inline' @keyup.enter.native='searchQuery'>
        <a-row :gutter='24'>
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='活动名称'>
              <a-input placeholder='请输入活动名称' v-model='queryParam.activityName'></a-input>
            </a-form-item>
          </a-col>
<!--          <a-col :xl='6' :lg='7' :md='8' :sm='24'>-->
<!--            <a-form-item label='组办方'>-->
<!--              <a-input placeholder='请输入组办方' v-model='queryParam.organizer'></a-input>-->
<!--            </a-form-item>-->
<!--          </a-col>-->
          <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <a-form-item label='活动类型'>
              <j-dict-select-tag placeholder='请选择状态' v-model='queryParam.activityType' dictCode='titer_type' />
              <!--              <a-input placeholder="请输入活动类型" v-model="queryParam.activityType"></a-input>-->
            </a-form-item>
          </a-col>

<!--          <a-col :xl='6' :lg='7' :md='8' :sm='24'>-->
<!--            <a-form-item label='活动状态'>-->
<!--              <j-dict-select-tag placeholder='请选择状态' v-model='queryParam.status' dictCode='titie_status' />-->
<!--              &lt;!&ndash;                <a-input placeholder="请输入活动状态" v-model="queryParam.status"></a-input>&ndash;&gt;-->
<!--            </a-form-item>-->
<!--          </a-col>-->
          <!--          <template v-if="toggleSearchStatus">-->

          <!--          </template>-->
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
      <!--      <a-button type="primary" icon="download" @click="handleExportXls('活动定制')">导出</a-button>-->
      <!--      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">-->
      <!--        <a-button type="primary" icon="import">导入</a-button>-->
      <!--      </a-upload>-->
      <!--      &lt;!&ndash; 高级查询区域 &ndash;&gt;-->
      <!--      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>-->
      <!--      <a-dropdown v-if="selectedRowKeys.length > 0">-->
      <!--        <a-menu slot="overlay">-->
      <!--          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>-->
      <!--        </a-menu>-->
      <!--        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>-->
      <!--      </a-dropdown>-->
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
                                     <a @click='handleDetail(record)'>详情</a>



          <span v-if='record.status==="1"'>
            <a-divider type='vertical' />
 <a @click='handleEdit(record)' >编辑</a>
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

    <bkb-activity-customization-modal ref='modalForm' @ok='modalFormOk'></bkb-activity-customization-modal>
    <bkb-activity-person23-modal ref='modalForm2' @ok='modalFormOk'></bkb-activity-person23-modal>
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import BkbActivityCustomizationModal from './modules/BkbActivityCustomizationModal'

import { getAction } from '@api/manage'
import BkbActivityPerson23Modal from '@views/mission/bkbCustomization/bkb_activity_person/BkbActivityPerson23Modal'

export default {
  name: 'BkbActivityCustomizationList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    BkbActivityCustomizationModal,
    BkbActivityPerson23Modal
  },
  data() {
    return {
      description: '活动定制管理页面',
      // 表头
      p_data: [],
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
          title: '活动名称',
          align: 'center',
          dataIndex: 'activityName'
        },
        {
          title: '活动类型',
          align: 'center',
          dataIndex: 'activityType_dictText'
        },
        {
          title: '活动封面图',
          align: 'center',
          dataIndex: 'coverUrl',
          scopedSlots: { customRender: 'imgSlot' }
        },
        // {
        //   title: '活动地址',
        //   align: 'center',
        //   dataIndex: 'activitySite'
        // },
        // {
        //   title: '组办方',
        //   align: 'center',
        //   dataIndex: 'organizer'
        // },
        // {
        //   title: '组办方电话',
        //   align: 'center',
        //   dataIndex: 'organizerPhone'
        // },
        // {
        //   title: '开始时间',
        //   align: 'center',
        //   dataIndex: 'startDate'
        // },
        // {
        //   title: '结束时间',
        //   align: 'center',
        //   dataIndex: 'endDate'
        // },
        {
          title: '活动订单',
          align: 'center',
          dataIndex: 'orderQuantity'
        },
        // {
        //   title: '活动状态',
        //   align: 'center',
        //   dataIndex: 'status_dictText'
        // },
        // {
        //   title: '活动图片',
        //   align: 'center',
        //   dataIndex: 'activityUrl',
        //   scopedSlots: { customRender: 'imgSlot' }
        // },
        // {
        //   title:'活动简介',
        //   align:"center",
        //   dataIndex: 'activityIntro'
        // },
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
        list: '/mission/bkbActivityCustomization/list',
        delete: '/mission/bkbActivityCustomization/delete',
        deleteBatch: '/mission/bkbActivityCustomization/deleteBatch',
        exportXlsUrl: '/mission/bkbActivityCustomization/exportXls',
        importExcelUrl: 'mission/bkbActivityCustomization/importExcel',
        list2: '/mission/bkbActivityCustomization/activityPerson'

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
      getAction(this.url.list2, { ids: record.id }).then(res => {
        if (res.success) {
          this.$refs.modalForm2.edit(res.result)
          this.$refs.modalForm2.title = '预约人数'
          this.$refs.modalForm2.disableSubmit = true
        }
      })
    },
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'activityName', text: '活动名称', dictCode: '' })
      fieldList.push({ type: 'int', value: 'activityType', text: '活动类型', dictCode: 'titer_type' })
      fieldList.push({ type: 'string', value: 'coverUrl', text: '活动封面图', dictCode: '' })
      fieldList.push({ type: 'string', value: 'activitySite', text: '活动地址', dictCode: '' })
      fieldList.push({ type: 'string', value: 'organizer', text: '组办方', dictCode: '' })
      fieldList.push({ type: 'string', value: 'organizerPhone', text: '组办方电话', dictCode: '' })
      fieldList.push({ type: 'datetime', value: 'startDate', text: '开始时间' })
      fieldList.push({ type: 'datetime', value: 'endDate', text: '结束时间' })
      fieldList.push({ type: 'int', value: 'orderQuantity', text: '活动订单', dictCode: '' })
      fieldList.push({ type: 'string', value: 'status', text: '活动状态', dictCode: 'titie_status' })
      fieldList.push({ type: 'string', value: 'activityUrl', text: '活动图片', dictCode: '' })
      fieldList.push({ type: 'string', value: 'activityIntro', text: '活动简介', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>