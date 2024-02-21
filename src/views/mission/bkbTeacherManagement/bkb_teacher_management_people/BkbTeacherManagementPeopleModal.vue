<template>
  <j-modal
    :title='title'
    :width='width'
    :visible='visible'
    switchFullscreen
    @ok='handleOk'
    :okButtonProps="{ class:{'jee-hidden': disableSubmit} }"
    @cancel='handleCancel'
    cancelText='关闭'>
    <div>

      <div class='table-page-search-wrapper'>
        <a-form layout='inline' @keyup.enter.native='searchQuery'>
          <a-row :gutter='24'>
            <a-col :xl='6' :lg='7' :md='8' :sm='24'>
              <a-form-item label='接单状态'>
                <j-dict-select-tag placeholder='请选择状态' v-model='queryParam.teacherStatus' dictCode='dan_status' />
              </a-form-item>
            </a-col>
            <a-col :xl='6' :lg='7' :md='8' :sm='24'>
            <span style='float: left;overflow: hidden;' class='table-page-search-submitButtons'>
              <a-button type='primary' @click='searchQuery' icon='search'>查询</a-button>
              <a-button type='primary' @click='searchReset' icon='reload' style='margin-left: 8px'>重置</a-button>
            </span>
            </a-col>
          </a-row>
        </a-form>
      </div>

      <a-table
        ref='table'
        size='middle'
        :scroll='{x:true}'
        bordered
        rowKey='id'
        :columns='columns'
        :dataSource='dataSource'
        :loading='loading'
        class='j-table-force-nowrap'>

        <span slot='action' slot-scope='text, record'>
                    <span v-if='record.teacherStatus==="0"'>
                            <a @click='handleDetail2(record)'>接单</a>

            <a-divider type='vertical' v-if='record.teacherStatus!=="0"' />
          </span>
                    <span v-if='record.teacherStatus!=="0"'>
                <a-popconfirm title='确定删除吗?' @confirm='() => handleDelete(record)'>
                  <a>删除</a>
                </a-popconfirm>
          </span>


        </span>


      </a-table>

      <bkb-teacher-management-2-modal ref='modalForm' @ok='modalFormOk'></bkb-teacher-management-2-modal>

    </div>
  </j-modal>
</template>

<script>


import BkbTeacherManagement2Modal
  from '@views/mission/bkbTeacherManagement/bkb_teacher_management_people/BkbTeacherManagement2Modal'
import { deleteAction, getAction } from '@api/manage'

export default {
  name: 'BkbTeacherManagementPeopleModal',
  components: { BkbTeacherManagement2Modal },
  // props: {
  //   dataSource: {
  //     type: Array,
  //     default: [],
  //     required: true
  //   }
  // },
  data() {
    return {
      queryParam: {},
      dataSource: [],
      loading: false,
      title: '',
      width: 800,
      visible: false,
      disableSubmit: false,
      description: '活动定制预约人数表管理页面',
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
          title: '预约人姓名',
          align: 'center',
          dataIndex: 'appointeePeople'
        },
        {
          title: '预约人电话',
          align: 'center',
          dataIndex: 'appointeePhone'
        },
        {
          title: '预约时间',
          align: 'center',
          dataIndex: 'appointeeTime'
        },
        {
          title: '创建日期',
          align: 'center',
          dataIndex: 'createTime'
        },
        {
          title: '状态',
          align: 'center',
          dataIndex: 'teacherStatus_dictText'
        },
        {
          title: '备注',
          align: 'center',
          dataIndex: 'remarks'
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
        edit: '/mission/bkbTeacherManagement/reservationViewingPut',
        delete: '/mission/bkbTeacherManagementPeople/delete',
        list2: '/mission/bkbTeacherManagementPeople/list'
      },
      mode: {},
      dictOptions: {},
      superFieldList: []
    }
  },
  methods: {
    searchQuery() {
      this.queryParam.teacherId = this.mode.id
      getAction(this.url.list2, this.queryParam).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
        }
      })
    },
    searchReset() {
      this.queryParam = {}
      // this.list_data(this.mode)
    },
    handleDetail2(v) {
      this.mode = Object.assign({}, v)
      this.$refs.modalForm.edit(v)
      this.$refs.modalForm.title = '确认成单'
      this.$refs.modalForm.visible = true
    },
    handleDelete(v) {
      deleteAction(this.url.delete, { id: v.id }).then(res => {
        if (res.success) {
          this.$message.success(res.message)
          this.list_data(v)
        }
      })
      // this.list_data(v)
    },
    list_data(v) {
      console.warn(v)
      getAction(this.url.list2, { teacherId: v ? v.teacherId : '0' }).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
        }
      })
      this.loading = false
    },
    modalFormOk() {
      // 新增/修改 成功时，重载列表
      this.list_data(this.mode)
      // this.loadData()
      // //清空列表选中
      // this.onClearSelected()
    },
    add() {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.realForm.add()
      })
    },
    edit(record) {
      this.visible = true
      this.loading = true
      this.list_data(record[0])
      this.loading = false
    },
    close() {
      this.$emit('close')
      this.visible = false
    },
    handleOk() {
      this.close()
      this.list_data(this.mode)
      // this.$refs.realForm.submitForm()
    },
    submitCallback() {
      this.$emit('ok')
      this.visible = false
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>