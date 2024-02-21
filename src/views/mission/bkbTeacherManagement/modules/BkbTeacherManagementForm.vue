<template>
  <a-spin :spinning='confirmLoading'>
    <j-form-container :disabled='formDisabled'>
      <a-form-model ref='form' :model='model' :rules='validatorRules' slot='detail'>
        <a-row>
          <a-col :span='24'>
            <a-form-model-item label='头像' :labelCol='labelCol' :wrapperCol='wrapperCol' prop='headImage'>
              <j-upload v-model="model.headImage" fileType="image" :number="1" fileDir="/TeacherManagement/img"></j-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span='24'>
            <a-form-model-item label='演讲老师名称' :labelCol='labelCol' :wrapperCol='wrapperCol' prop='speechTeacher'>
              <a-input v-model='model.speechTeacher' placeholder='请输入演讲老师名称'></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span='24'>
            <a-form-model-item label='演讲年限' :labelCol='labelCol' :wrapperCol='wrapperCol' prop='speechYear'>
              <a-input-number v-model='model.speechYear' placeholder='请输入演讲年限' style='width: 100%' />
            </a-form-model-item>
          </a-col>
          <a-col :span='24'>
            <a-form-model-item label='演讲级别' :labelCol='labelCol' :wrapperCol='wrapperCol' prop='speechLevel'>
              <j-dict-select-tag type='list' v-model='model.speechLevel' dictCode='speech_level'
                                 placeholder='请选择演讲级别' />
            </a-form-model-item>
          </a-col>
          <a-col :span='24'>
            <a-form-model-item label='演讲费用' :labelCol='labelCol' :wrapperCol='wrapperCol' prop='speechExpenses'>
              <a-input v-model='model.speechExpenses' placeholder='请输入演讲费用'></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span='24'>
            <a-form-model-item label='联系电话' :labelCol='labelCol' :wrapperCol='wrapperCol' prop='phone'>
              <a-input v-model='model.phone' placeholder='请输入联系电话'></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span='24'>
            <a-form-model-item label='图片上传' :labelCol='labelCol' :wrapperCol='wrapperCol' prop='uploadPictures'>
              <j-upload v-model="model.uploadPictures" fileType="image" :number="5" fileDir="/TeacherManagement/img"></j-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span='24'>
            <a-form-model-item label='个人简介' :labelCol='labelCol' :wrapperCol='wrapperCol' prop='personalProfile'>
              <a-textarea placeholder='请输入个人简介' v-model='model.personalProfile' />
            </a-form-model-item>
          </a-col>
          <!--          <a-col :span="24">-->
          <!--            <a-form-model-item label="预约订单" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="bookingOrder">-->
          <!--              <a-input-number v-model="model.bookingOrder" placeholder="请输入预约订单" style="width: 100%" />-->
          <!--            </a-form-model-item>-->
          <!--          </a-col>-->
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>

import { httpAction, getAction } from '@/api/manage'
import { validateDuplicateValue } from '@/utils/util'
import { initDictOptions } from '@comp/dict/JDictSelectUtil'

export default {
  name: 'BkbTeacherManagementForm',
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
        headImage: [
          { required: true, message: '请输入头像!' }
        ],
        speechTeacher: [
          { required: true, message: '请输入演讲老师名称!' }
        ],
        speechYear: [
          { required: true, message: '请输入演讲年限!' }
        ],
        speechLevel: [
          { required: true, message: '请输入演讲级别!' }
        ],
        speechExpenses: [
          { required: true, message: '请输入演讲费用!' }
        ],
        phone: [
          { required: true, message: '请输入联系电话!' },
          { pattern: /^1[3456789]\d{9}$/, message: '请输入正确的手机号码!' }
        ],
        uploadPictures: [
          { required: true, message: '请输入图片上传!' }
        ],
        personalProfile: [
          { required: true, message: '请输入个人简介!' }
        ]
      },
      url: {
        add: '/mission/bkbTeacherManagement/add',
        edit: '/mission/bkbTeacherManagement/edit',
        queryById: '/mission/bkbTeacherManagement/queryById'
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
      this.model = Object.assign({}, record)
      // this.model.speechLevel =this.model.speechLevel ? this.model.speechLevel : 0
      this.visible = true
    },
    submitForm() {
      const that = this
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }
          httpAction(httpurl, this.model, method).then((res) => {
            if (res.success) {
              that.$message.success(res.message)
              that.$emit('ok')
            } else {
              that.$message.warning(res.message)
            }
          }).finally(() => {
            that.confirmLoading = false
          })
        }

      })
    }
  }
}
</script>