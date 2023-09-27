<!--关联功能-->
<!--自定义关联功能css style 属性-->
<template>
  <el-form-item :label="i18nt('designer.setting.rfCssStyle')">
    <el-input @change="handlerChange" v-model="optionModel.rfCssStyle"></el-input>
  </el-form-item>
</template>

<script>
import i18n from "@/utils/i18n"

export default {
  name: "rfCssStyle-editor",
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

    },
    handlerSetCriteria() {

    },
  }
}
</script>

<style scoped>

</style>
