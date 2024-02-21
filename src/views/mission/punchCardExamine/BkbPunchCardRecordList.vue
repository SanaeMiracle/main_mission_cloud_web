<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="打卡用户手机号">
              <a-input placeholder="请输入打卡用户手机号" v-model="queryParam.punchCardPhone"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="打卡类型">
              <a-select allowClear placeholder="请选择打卡类型" v-model="queryParam.punchCardType">
                <a-select-option value="厨余分类">厨余分类</a-select-option>
                <a-select-option value="建言献策">建言献策</a-select-option>
                <a-select-option value="分类随手拍">分类随手拍</a-select-option>
                <a-select-option value="分类活动/培训参与">分类活动/培训参与</a-select-option>
                <a-select-option value="光盘行动">光盘行动</a-select-option>
                <a-select-option value="可回收物回收">可回收物回收</a-select-option>
                <a-select-option value="有害垃圾回收">有害垃圾回收</a-select-option>
                <a-select-option value="21天打卡">21天打卡</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="审核类型">
              <a-select allowClear placeholder="请选择审核类型" v-model="queryParam.oneClickReviewStatus">
                <a-select-option value="AUTOMATED_MODERATION">自动审核</a-select-option>
                <a-select-option value="MANUAL_REVIEW">手动审核</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="审核状态">
              <a-select allowClear placeholder="请选择审核状态" v-model="queryParam.reviewStatus">
                <a-select-option value="TO_BE_REVIEWED">待审核</a-select-option>
                <a-select-option value="REVIEWED">已审核（通过）</a-select-option>
                <a-select-option value="FAILED">已审核（未通过）</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="36" :span="40">
            <a-form-item label="打卡时间">
              <!-- <j-date
                :show-time="true"
                date-format="YYYY-MM-DD HH:mm:ss"
                placeholder="请选择发布时间"
                v-model="queryParam.updateTime"
              ></j-date> -->
              <a-range-picker
                style="width: 320px;"
                :show-time="{ format: 'HH:mm:ss' }"
                date-format="YYYY-MM-DD HH:mm:ss"
                format="YYYY-MM-DD HH:mm:ss"
                :placeholder="['开始时间', '结束时间']"
                v-model="queryParam.timeRange"
                @change="handleTimePickerChange"
              />
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->

    <!-- table区域-begin -->
    <div>
      <a-table
        ref="table"
        size="middle"
        :scroll="{ x: true }"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        class="j-table-force-nowrap"
        @change="handleTableChange"
      >
        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text, record">
          <div v-if="!text" style="height:30px;line-height: 30px; font-size: 12px;font-style: italic;">无图片</div>
          <div v-else style="display: flex; justify-content: space-around; align-items: center;">
            <div v-for="item in text ? text.split(',') : ''">
              <img
                :src="getImgView(item)"
                :preview="record.id"
                alt=""
                style="height:30px; font-size: 12px;font-style: italic;"
              />
            </div>
          </div>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button v-else :ghost="true" type="primary" icon="download" size="small" @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <span v-if="record.reviewStatus == 'TO_BE_REVIEWED'">
            <a @click="handleExamine(record)">审核</a>

            <a-divider type="vertical" />
          </span>
          <a @click="handleDetail(record)">详情</a>
          <!-- <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
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
          </a-dropdown> -->
        </span>
      </a-table>
    </div>
    <a-modal :width="600" v-model:visible="visible" title="审核" @ok="handleOk">
      <template #footer>
        <a-button key="back" @click="visible = false">取消</a-button>
        <a-button key="submit" type="primary" :loading="loading" @click="handleOk">提交</a-button>
      </template>
      <j-form-container :disabled="true">
        <a-form-model :model="model" slot="detail">
          <a-row>
            <a-col :span="24">
              <a-form-model-item
                label="打卡用户手机号"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="punchCardPhone"
              >
                <a-input v-model="model.punchCardPhone" placeholder="请输入打卡用户手机号"></a-input>
              </a-form-model-item>
            </a-col>
            <a-col :span="24">
              <a-form-model-item label="打卡类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="punchCardType">
                <a-select allowClear placeholder="请选择打卡类型" v-model="model.punchCardType">
                  <a-select-option value="厨余分类">厨余分类</a-select-option>
                  <a-select-option value="建言献策">建言献策</a-select-option>
                  <a-select-option value="分类随手拍">分类随手拍</a-select-option>
                  <a-select-option value="分类活动/培训参与">分类活动/培训参与</a-select-option>
                  <a-select-option value="光盘行动">光盘行动</a-select-option>
                  <a-select-option value="可回收物回收">可回收物回收</a-select-option>
                  <a-select-option value="有害垃圾回收">有害垃圾回收</a-select-option>
                  <a-select-option value="21天打卡">21天打卡</a-select-option>
                </a-select>
              </a-form-model-item>
            </a-col>
            <a-col :span="24">
              <a-form-model-item
                label="打卡描述"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="punchCardDescribe"
              >
                <a-input v-model="model.punchCardDescribe" placeholder="请输入打卡描述"></a-input>
              </a-form-model-item>
            </a-col>
            <a-col :span="24">
              <a-form-model-item
                label="打卡积分"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="punchCardIntegral"
              >
                <a-input-number v-model="model.punchCardIntegral" placeholder="请输入打卡积分" style="width: 100%" />
              </a-form-model-item>
            </a-col>
            <a-col :span="24">
              <a-form-model-item
                label="打卡图片"
                :labelCol="labelCol"
                :wrapperCol="wrapperCol"
                prop="punchCardArrayUrl"
              >
                <!-- <a-input v-model="model.punchCardArrayUrl" placeholder="请输入打卡图片"></a-input> -->
                <j-upload
                  v-model="model.punchCardArrayUrl"
                  fileType="image"
                  :number="9"
                  fileDir="/ActivityCustomization/img"
                ></j-upload>
              </a-form-model-item>
            </a-col>

            <a-col :span="24">
              <a-form-model-item label="打卡时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="createTime">
                <a-input v-model="model.createTime" placeholder="请输入打卡位置"></a-input>
              </a-form-model-item>
            </a-col>
          </a-row>
        </a-form-model>
      </j-form-container>
      <a-divider />
      <a-form-model
        ref="examineForm"
        :model="examineModel"
        :rules="{ reviewStatus: [{ required: true, message: '请选择是否审核通过!' }] }"
      >
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="是否审核通过" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="reviewStatus">
              <a-radio-group v-model="examineModel.reviewStatus" placeholder="请选择是否审核通过">
                <a-radio value="REVIEWED">通过</a-radio>
                <a-radio value="FAILED">不通过</a-radio>
              </a-radio-group>
            </a-form-model-item>
          </a-col>

          <a-col :span="24" v-if="examineModel.reviewStatus != 'REVIEWED'">
            <a-form-model-item label="审核未通过原因" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="failedRemark">
              <a-input type="textarea" v-model="examineModel.failedRemark" placeholder="请输入审核未通过原因"></a-input>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </a-modal>
    <bkb-punch-card-record-modal ref="modalForm" @ok="modalFormOk"></bkb-punch-card-record-modal>
  </a-card>
</template>

<script>
import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import BkbPunchCardRecordModal from './modules/BkbPunchCardRecordModal'
import { httpAction, getAction } from '@/api/manage'
export default {
  name: 'BkbPunchCardRecordList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    BkbPunchCardRecordModal
  },
  data() {
    return {
      model: {},
      examineModel: {},
      loading: false,
      visible: false,
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      description: 'bkb_punch_card_record管理页面',
      // 表头
      columns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 40,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '打卡用户手机号',
          align: 'center',
          dataIndex: 'punchCardPhone'
        },
        {
          title: '打卡描述',
          align: 'center',
          dataIndex: 'punchCardDescribe',
          width: 200
        },
        {
          title: '打卡积分',
          align: 'center',
          dataIndex: 'punchCardIntegral'
        },
        {
          title: '打卡图片',
          align: 'center',
          dataIndex: 'punchCardArrayUrl',
          scopedSlots: { customRender: 'imgSlot' }
        },
        {
          title: '打卡类型',
          align: 'center',
          dataIndex: 'punchCardType'
        },
        {
          title: '审核类型',
          align: 'center',
          dataIndex: 'oneClickReviewStatus',
          customRender: function(t, r, index) {
            if (t == 'AUTOMATED_MODERATION') return '自动审核'
            else if (t == 'MANUAL_REVIEW') return '手动审核'
            else return ''
          }
        },
        {
          title: '审核状态',
          align: 'center',
          dataIndex: 'reviewStatus',
          customRender: function(t, r, index) {
            switch (t) {
              case 'TO_BE_REVIEWED':
                return '待审核'
              case 'REVIEWED':
                return '已审核（通过）'
              case 'FAILED':
                return '已审核（未通过）'
              default:
                return t
            }
          }
        },
        {
          title: '打卡位置',
          align: 'center',
          dataIndex: 'punchCardAddress'
        },
        {
          title: '打卡时间',
          align: 'center',
          dataIndex: 'createTime'
        },
        {
          title: '审核未通过原因',
          align: 'center',
          dataIndex: 'failedRemark'
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
        list: '/mission/bkbPunchCardRecord/auditLists',
        delete: '/mission/bkbPunchCardRecord/delete',
        deleteBatch: '/mission/bkbPunchCardRecord/deleteBatch',
        exportXlsUrl: '/mission/bkbPunchCardRecord/exportXls',
        importExcelUrl: 'mission/bkbPunchCardRecord/importExcel'
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
    handleExamine(e) {
      this.visible = true
      this.model = e
    },
    handleOk() {
      // 触发表单验证
      this.$refs.examineForm.validate(valid => {
        if (valid) {
          let data = {
            remark: this.examineModel.failedRemark,
            status: this.examineModel.reviewStatus,
            id: this.model.id
          }
          let httpurl = 'mission/bkbPunchCardRecord/auditOne'
          let method = 'GET'
          getAction(httpurl, data)
            .then(res => {
              if (res.success) {
                this.$message.success(res.message)
              } else {
                this.$message.warning(res.message)
              }
            })
            .finally(() => {
              this.visible = false
              this.reset()
            })
        }
      })
    },
    handleTimePickerChange(e) {
      // delete this.queryParam.timeRange
      this.queryParam.startDate = e.length > 0 ? this.getTime(e[0]._d) : ''
      this.queryParam.endDate = e.length > 0 ? this.getTime(e[1]._d) : ''
    },
    reset() {
      this.queryParam.startDate = ''
      this.queryParam.endDate = ''
      this.queryParam.timeRange = []
      this.searchReset()
    },
    getTime(date) {
      let y = date.getFullYear()
      let m = date.getMonth() + 1
      m = m < 10 ? '0' + m : m
      let d = date.getDate()
      d = d < 10 ? '0' + d : d
      let h = date.getHours()
      h = h < 10 ? '0' + h : h
      let M = date.getMinutes()
      M = M < 10 ? '0' + M : M
      let s = date.getSeconds()
      s = s < 10 ? '0' + s : s
      return y + '-' + m + '-' + d + ' ' + h + ':' + M + ':' + s
    },
    handlePublish(e) {
      httpAction('/czq/bkbListManage/edit', { ...e, status: '已发布' }, 'put')
        .then(res => {
          if (res.success) {
            this.$message.success(res.message)
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          this.modalFormOk()
        })
    },
    initDictConfig() {},
    getSuperFieldList() {
      let fieldList = []
      fieldList.push({ type: 'string', value: 'punchCardPhone', text: '打卡用户手机号', dictCode: '' })
      fieldList.push({ type: 'string', value: 'punchCardDescribe', text: '打卡描述', dictCode: '' })
      fieldList.push({ type: 'int', value: 'punchCardIntegral', text: '打卡积分', dictCode: '' })
      fieldList.push({ type: 'string', value: 'punchCardArrayUrl', text: '打卡图片', dictCode: '' })
      fieldList.push({ type: 'string', value: 'punchCardType', text: '打卡类型', dictCode: '' })
      fieldList.push({ type: 'string', value: 'oneClickReviewStatus', text: '一键审核状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'isClickReviewStatus', text: '是否需要审核', dictCode: '' })
      fieldList.push({ type: 'string', value: 'isSynchronousCircle', text: '是否同步圈子', dictCode: '' })
      fieldList.push({ type: 'string', value: 'reviewStatus', text: '审核状态', dictCode: '' })
      fieldList.push({ type: 'string', value: 'punchCardAddress', text: '打卡位置', dictCode: '' })
      fieldList.push({ type: 'string', value: 'failedRemark', text: '未通过备注', dictCode: '' })
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
