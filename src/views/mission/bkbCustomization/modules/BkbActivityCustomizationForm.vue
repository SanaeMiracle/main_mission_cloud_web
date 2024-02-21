<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="活动名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="activityName">
              <a-input v-model="model.activityName" placeholder="请输入活动名称"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="活动类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="activityType">

              <j-dict-select-tag type="list" v-model="model.activityType" dictCode="titer_type" placeholder="请选择活动类型" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="活动封面图" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="coverUrl">
              <j-upload v-model="model.coverUrl" fileType="image" :number="1" fileDir="/ActivityCustomization/img"></j-upload>
            </a-form-model-item>
          </a-col>
<!--          <a-col :span="24">-->
<!--            <a-form-model-item label="活动地址" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="activitySite">-->
<!--              <a-input v-model="model.activitySite" placeholder="请输入活动地址"  ></a-input>-->
<!--            </a-form-model-item>-->
<!--          </a-col>-->
<!--          <a-col :span="24">-->
<!--            <a-form-model-item label="组办方" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="organizer">-->
<!--              <a-input v-model="model.organizer" placeholder="请输入组办方"  ></a-input>-->
<!--            </a-form-model-item>-->
<!--          </a-col>-->
<!--          <a-col :span="24">-->
<!--            <a-form-model-item label="组办方电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="organizerPhone">-->
<!--              <a-input v-model="model.organizerPhone" placeholder="请输入组办方电话"  ></a-input>-->
<!--            </a-form-model-item>-->
<!--          </a-col>-->
<!--          <a-col :span="24">-->
<!--            <a-form-model-item label="开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="startDate">-->
<!--              <j-date placeholder="请选择开始时间"  v-model="model.startDate" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%" />-->
<!--            </a-form-model-item>-->
<!--          </a-col>-->
<!--          <a-col :span="24">-->
<!--            <a-form-model-item label="结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="endDate">-->
<!--              <j-date placeholder="请选择结束时间"  v-model="model.endDate" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%" />-->
<!--            </a-form-model-item>-->
<!--          </a-col>-->
<!--          <a-col :span="24">-->
<!--            <a-form-model-item label="活动时长" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="endDate">-->
<!--{{ time}}-->
<!--            </a-form-model-item>-->
<!--          </a-col>-->
<!--          <a-col :span="24">-->
<!--            <a-form-model-item label="活动订单" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="orderQuantity">-->
<!--              <a-input-number v-model="model.orderQuantity" placeholder="请输入活动订单" style="width: 100%" />-->
<!--            </a-form-model-item>-->
<!--          </a-col>-->
<!--          <a-col :span="24">-->
<!--            <a-form-model-item label="活动状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="status">-->
<!--              <j-dict-select-tag type="list" v-model="model.status" dictCode="titie_status" placeholder="请选择活动状态" />-->
<!--            </a-form-model-item>-->
<!--          </a-col>-->
          <a-col :span="24">
            <a-form-model-item label="活动简介" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="activityIntro">
              <a-textarea placeholder='请输入活动简介' v-model='model.activityIntro' />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="活动图片" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="activityUrl">
              <j-upload v-model="model.activityUrl" fileType="image" :number="5" fileDir="/ActivityCustomization/img"></j-upload>

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
    name: 'BkbActivityCustomizationForm',
    components: {
    },
    props: {
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {

        model:{
         },
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
           activityName: [
              { required: true, message: '请输入活动名称!'},
           ],
           activityType: [
              { required: true, message: '请输入活动类型!'},
           ],
           coverUrl: [
              { required: true, message: '请输入活动封面图!'},
           ],
           activitySite: [
              { required: true, message: '请输入活动地址!'},
           ],
           organizer: [
              { required: true, message: '请输入组办方!'},
           ],
           organizerPhone: [
              { required: true, message: '请输入组办方电话!'},
             { pattern: /^1[3456789]\d{9}$/, message: '请输入正确的手机号码!'},
           ],
           startDate: [
              { required: true, message: '请输入开始时间!'},
           ],
           endDate: [
              { required: true, message: '请输入结束时间!'},
           ],
           // orderQuantity: [
           //    { required: true, message: '请输入活动订单!'},
           // ],
           status: [
              { required: true, message: '请输入活动状态!'},
           ],
           activityUrl: [
              { required: true, message: '请输入活动图片!'},
           ],
           activityIntro: [
              { required: true, message: '请输入活动简介!'},
           ],
        },
        url: {
          add: "/mission/bkbActivityCustomization/add",
          edit: "/mission/bkbActivityCustomization/edit",
          queryById: "/mission/bkbActivityCustomization/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        return this.disabled
      },

      time() {
        if (this.model.endDate && this.model.startDate) {

          var difftime = (new Date(this.model.endDate) - new Date(this.model.startDate)) / 1000; //计算时间差
          console.log(difftime)

          var days = parseInt(difftime / 86400); // 天  24*60*60*1000
          var hours = parseInt(difftime / 3600) - 24 * days;    // 小时 60*60 总小时数-过去的小时数=现在的小时数
          var minutes = parseInt(difftime % 3600 / 60); // 分钟 -(day*24) 以60秒为一整份 取余 剩下秒数 秒数/60 就是分钟数
          var seconds = parseInt(difftime % 60);  // 以60秒为一整份 取余 剩下秒数


          return days + '天' + hours + '时' + minutes + '分' + seconds + '秒'
        }else{
          return ''
        }
      }
    },
    created () {
       //备份model原始值
      this.modelDefault = JSON.parse(JSON.stringify(this.model));
    },


    methods: {

      add () {
        this.edit(this.modelDefault);
      },
      edit (record) {
        this.model = Object.assign({}, record);
        this.visible = true;
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.$refs.form.validate(valid => {
          if (valid) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            httpAction(httpurl,this.model,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }

        })
      },
    }
  }
</script>