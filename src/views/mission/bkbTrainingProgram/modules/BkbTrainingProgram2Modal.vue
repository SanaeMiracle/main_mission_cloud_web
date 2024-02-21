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

    <a-descriptions title='演讲基础信息' bordered>
      <a-descriptions-item label='演讲人' :span='24'>
        {{ model.speakerId_dictText }}
      </a-descriptions-item>
      <a-descriptions-item label='培训时长'>
        {{ model.duration }}小时
      </a-descriptions-item>
      <a-descriptions-item label='反馈人数'>
        {{ model.people }}
      </a-descriptions-item>
    </a-descriptions>
    <a-divider orientation='left'>
      打卡图片
    </a-divider>
    <j-image-upload isMultiple disabled='true' v-model='model.picURL'></j-image-upload>

    <a-divider orientation='left'>
      反馈人
    </a-divider>

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
    </a-table>

  </j-modal>
</template>

<script>


import { getAction } from '@api/manage'

export default {
  name: 'BkbTrainingProgram2Modal',
  components: {},
  data() {
    return {
      title: '',
      width: 1000,
      visible: false,
      disableSubmit: false,
      model: {},
      dataSource: [],
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
          title: '评论人名称',
          align: 'center',
          dataIndex: 'userName'
        },
        {
          title: '获得积分',
          align: 'center',
          dataIndex: 'grade'
        },
        {
          title: '活动效果如何（1.5分)',
          align: 'center',
          dataIndex: 'grade1'
        },
        {
          title: '本次活动是否有用（1.5分）',
          align: 'center',
          dataIndex: 'grade2'
        },
        {
          title: '本次活动是否有用（1.5分）',
          align: 'center',
          dataIndex: 'grade3'
        },
        {
          title: '具他建议',
          align: 'center',
          dataIndex: 'comment'
        }
      ],
      url: {
        list: '/mission/bkbTrainingProgram/appointLookX',
        rsdata: '/mission/bkbTrainingProgram/appointLookS'
      }
    }
  },
  mounted() {

  },
  methods: {
    add() {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.realForm.add()
      })
    },
    edit(record) {
      console.error(record)
      this.model = { ...record }
      getAction(this.url.list, { id: this.model.id }).then(res => {
        if (res.success) {
          let array = []
          array = res.result.records
          this.dataSource = array.map(v => ({ ...v.bkbNewComment, grade: v.grade }))
        }
      })
      getAction(this.url.rsdata, { id: this.model.id }).then(res => {
        if (res.success) {
          this.model.people = res.result.people
          this.model.picURL = res.result.picURL
        }
      })

      this.visible = true
      // this.$nextTick(()=>{
      //   this.$refs.realForm.edit(record);
      // })
    },
    close() {
      this.$emit('close')
      this.visible = false
    },
    handleOk() {
      // this.$refs.realForm.submitForm();
      this.close()
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