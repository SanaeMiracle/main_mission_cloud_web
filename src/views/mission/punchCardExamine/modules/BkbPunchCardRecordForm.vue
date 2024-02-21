<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
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
            <a-form-model-item label="打卡描述" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="punchCardDescribe">
              <a-input type="textarea" v-model="model.punchCardDescribe" placeholder="请输入打卡描述"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="打卡积分" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="punchCardIntegral">
              <a-input-number v-model="model.punchCardIntegral" placeholder="请输入打卡积分" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="打卡图片" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="punchCardArrayUrl">
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
              label="审核类型"
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              prop="oneClickReviewStatus"
            >
              <a-select allowClear placeholder="请选择审核类型" v-model="model.oneClickReviewStatus">
                <a-select-option value="AUTOMATED_MODERATION">自动审核</a-select-option>
                <a-select-option value="MANUAL_REVIEW">手动审核</a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="审核状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="reviewStatus">
              <a-select allowClear placeholder="请选择审核状态" v-model="model.reviewStatus">
                <a-select-option value="TO_BE_REVIEWED">待审核</a-select-option>
                <a-select-option value="REVIEWED">已审核（通过）</a-select-option>
                <a-select-option value="FAILED">已审核（未通过）</a-select-option>
              </a-select>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="打卡位置" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="punchCardAddress">
              <a-input v-model="model.punchCardAddress" placeholder="请输入打卡位置"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="未通过备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="failedRemark">
              <a-input type="textarea" v-model="model.failedRemark" placeholder="请输入未通过备注"></a-input>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>
import { httpAction, getAction } from '@/api/manage'
import { validateDuplicateValue } from '@/utils/util'

export default {
  name: 'BkbPunchCardRecordForm',
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
      validatorRules: {},
      url: {
        add: '/mission/bkbPunchCardRecord/add',
        edit: '/mission/bkbPunchCardRecord/edit',
        queryById: '/mission/bkbPunchCardRecord/queryById'
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
          httpAction(httpurl, this.model, method)
            .then(res => {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
              } else {
                that.$message.warning(res.message)
              }
            })
            .finally(() => {
              that.confirmLoading = false
            })
        }
      })
    }
  }
}
</script>
