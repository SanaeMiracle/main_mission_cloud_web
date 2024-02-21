<template>
  <a-spin :spinning='confirmLoading'>
    <j-form-container :disabled='formDisabled'>
      <a-form-model ref='form' :model='model' :rules='validatorRules' slot='detail'>
        <a-row>
          <a-col :span='24'>
            <a-form-model-item label='订单状态' :labelCol='labelCol' :wrapperCol='wrapperCol' prop='teacherStatus'>
<!--              <a-input v-model='model.teacherStatus' placeholder='请选择类型'></a-input>-->
              <j-dict-select-tag placeholder='请选择状态' v-model='model.teacherStatus' dictCode='order_taking_status' />
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
  name: 'BkbTeacherManagement2Form',
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
        teacherStatus: [
          { required: true, message: '请选择下拉!' }
        ]
      },
      url: {
        edit: '/mission/bkbTeacherManagement/reservationViewingPut',
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
      record.teacherStatus = null
      this.model = Object.assign({}, record)
      this.visible = true
    },
    submitForm() {
      const that = this;
      let data = Object.assign({}, this.model)
      getAction(this.url.edit, {
        id: data.id,
        status: data.teacherStatus + '',
        remarks: data.remarks
      }).then(res => {
        if (res.success) {
          that.$message.success(res.message)
          that.$emit('ok')
        }
      })
      // 触发表单验证
      // this.$refs.form.validate(valid => {
      //   if (valid) {
      //     that.confirmLoading = true
      //     let httpurl = ''
      //     let method = ''
      //     if (!this.model.id) {
      //       httpurl += this.url.add
      //       method = 'post'
      //     } else {
      //       httpurl += this.url.edit
      //       method = 'put'
      //     }
      //     httpAction(httpurl, this.model, method).then((res) => {
      //       if (res.success) {
      //         that.$message.success(res.message)
      //         that.$emit('ok')
      //       } else {
      //         that.$message.warning(res.message)
      //       }
      //     }).finally(() => {
      //       that.confirmLoading = false
      //     })
      //   }
      //
      // })
    }
  }
}
</script>