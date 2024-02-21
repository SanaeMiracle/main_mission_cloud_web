<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="培训标题" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="title">
              <a-input v-model="model.title" placeholder="请输入培训标题"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="封面" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="coverUrl">
              <j-upload v-model="model.coverUrl" fileType="image" :number="1" fileDir="/TrainingProgram/img"></j-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="培训类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="type">
              <j-dict-select-tag type="list" v-model="model.type" dictCode="train_type" placeholder="请选择培训类型" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="培训地址" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="addr">
              <a-input v-model="model.addr" placeholder="请输入培训地址"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="startDate">
              <j-date placeholder="请选择开始时间"  v-model="model.startDate" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="培训时长" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="duration">
              <a-input-number v-model="model.duration" placeholder="请输入培训时长" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="获得积分" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="integral">
              <a-input-number v-model="model.integral" placeholder="请输入获得积分" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="培训演讲人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="speakerId">
<!--              <j-dict-select-tag type="list" v-model="model.speakerId" dictCode="" placeholder="请选择培训演讲人" />-->
              <j-dict-select-tag v-model='model.speakerId'
                                 :dictCode="`bkb_training_speakers,speaker,id,`"
                                 placeholder='请选择' />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="培训内容简介" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="content">
              <a-input v-model="model.content" placeholder="请输入培训内容简介"  ></a-input>
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
    name: 'BkbTrainingProgramForm',
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
           title: [
              { required: true, message: '请输入培训标题!'},
           ],
           coverUrl: [
              { required: true, message: '请输入封面!'},
           ],
           addr: [
              { required: true, message: '请输入培训地址!'},
           ],
           startDate: [
              { required: true, message: '请输入开始时间!'},
           ],
           duration: [
              { required: true, message: '请输入培训时长!'},
           ],
           speakerId: [
              { required: true, message: '请输入培训演讲人!'},
           ],
          content: [
            { required: true, message: '请输入培训简介!'},
          ],
          type: [
            { required: true, message: '请选择培训类型!'},
          ],
        },
        url: {
          add: "/mission/bkbTrainingProgram/add",
          edit: "/mission/bkbTrainingProgram/edit",
          queryById: "/mission/bkbTrainingProgram/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        return this.disabled
      },
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