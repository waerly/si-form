<!--关联功能-->
<template>
  <el-form-item :label="i18nt('designer.setting.rfCustomCol')">
    <el-switch @change="handlerChange" v-model="optionModel.rfCustomCol"></el-switch>
  </el-form-item>
</template>

<script>
import i18n from "@/utils/i18n"

export default {
  name: "rfCustomCol-editor",
  mixins: [i18n],
  props: {
    designer: Object,
    selectedWidget: Object,
    optionModel: Object,
  },
  methods: {

    // eslint-disable-next-line no-unused-vars
    handlerChange(val) {
      // eslint-disable-next-line no-prototype-builtins

    }
  }
}
</script>

<style scoped>

</style>
