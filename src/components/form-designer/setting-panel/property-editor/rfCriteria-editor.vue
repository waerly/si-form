<!--关联功能-->
<template>
  <el-form-item :label="i18nt('designer.setting.rfCriteria')">
    <el-input @change="handlerChange" v-model="optionModel.rfCriteria"></el-input>
    <el-button type="info" @click="handlerSetCriteria">
      {{i18nt('designer.hint.widgetSetting')}}</el-button>
  </el-form-item>
</template>

<script>
import i18n from "@/utils/i18n"

export default {
  name: "rfCriteria-editor",
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
