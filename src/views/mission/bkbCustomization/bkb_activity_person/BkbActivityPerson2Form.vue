<template>
  <a-spin :spinning='confirmLoading'>
    <j-form-container :disabled='formDisabled'>
      <a-form-model ref='form' :model='model' :rules='validatorRules' slot='detail'>
        <a-row>
          <a-col :span='24'>
            <a-form-model-item label='订单状态' :labelCol='labelCol' :wrapperCol='wrapperCol' prop='bookingStatus'>
              <!--              <a-input v-model='model.teacherStatus' placeholder='请选择类型'></a-input>-->
              <j-dict-select-tag placeholder='请选择状态' v-model='model.bookingStatus' dictCode='order_taking_status' />
            </a-form-model-item>
          </a-col>
          <a-col :span='24'>
            <a-form-model-item label='备注' :labelCol='labelCol' :wrapperCol='wrapperCol'>
              <a-textarea placeholder='请输入备注' v-model='model.remarks' />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>

import { httpAction, getAction } from '@/api/manage'

export default {
  name: 'BkbActivityPerson2Form',
  components: {},
  props: {
    //表单禁用
    disabled: {
      type: Boolean,
      default: false,
      required: false
    }
  },
  data() {
    return {
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
      validatorRules: {
        bookingStatus: [
          { required: true, message: '请选择下拉!' }
        ]
      },
      url: {
        edit: '/mission/bkbActivityCustomization/reservationViewingPut',
        delete: '/mission/bkbTeacherManagementPeople/delete'
      }
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model))
  },
  methods: {
    add() {
      this.edit(this.modelDefault)
    },
    edit(record) {
      record.bookingStatus = null
      this.model = record
      this.visible = true
    },
    submitForm() {
      const that = this
      let data = Object.assign({}, this.model)
      getAction(this.url.edit, {
        id: data.id,
        status: data.bookingStatus + '',
        remarks: data.remarks
      }).then(res => {
        if (res.success) {
          that.$message.success(res.message)
          that.$emit('ok')
        }
      })
    }
  }
}
</script>