<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    switchFullscreen
    @ok="handleOk"
    :okButtonProps="{ class:{'jee-hidden': disableSubmit} }"
    @cancel="handleCancel"
    cancelText="关闭">
    <bkb-teacher-management-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></bkb-teacher-management-form>
  </j-modal>
</template>

<script>

  import BkbTeacherManagementForm from './BkbTeacherManagement2Form'
  export default {
    name: 'BkbTeacherManagement2Modal',
    components: {
      BkbTeacherManagementForm
    },
    data () {
      return {
        title:'',
        width:800,
        visible: false,
        disableSubmit: false
      }
    },
    methods: {
      add () {
        this.visible=true
        this.$nextTick(()=>{
          this.$refs.realForm.add();
        })
      },
      edit (record) {
        this.visible=true
        this.$nextTick(()=>{
          this.$refs.realForm.edit(record);
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        this.$refs.realForm.submitForm();
      },
      submitCallback(){
        this.visible = false;
        this.$emit('ok');
      },
      handleCancel () {
        this.close()
      }
    }
  }
</script>