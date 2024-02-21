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
    <bkb-activity-person2-form ref='realForm' @ok='submitCallback'
                               :disabled='disableSubmit'></bkb-activity-person2-form>
  </j-modal>
</template>

<script>

import BkbActivityPerson2Form from '@views/mission/bkbCustomization/bkb_activity_person/BkbActivityPerson2Form'

export default {
  name: 'BkbActivityPerson2Modal',
  components: {
    BkbActivityPerson2Form
  },
  data() {
    return {
      title: '',
      width: 800,
      visible: false,
      disableSubmit: false
    }
  },
  methods: {
    add() {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.realForm.add()
      })
    },
    edit(record) {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.realForm.edit(record)
      })
    },
    close() {
      this.$emit('close')
      this.visible = false
    },
    handleOk() {
      this.$refs.realForm.submitForm()
    },
    submitCallback() {
      this.visible = false
      this.$emit('ok')
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>