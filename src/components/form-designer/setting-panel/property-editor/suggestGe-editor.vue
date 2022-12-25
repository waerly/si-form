<template>
  <el-form-item :label="i18nt('designer.setting.suggestGe')">
    <el-switch @change="handlerChange" v-model="optionModel.suggestGe"></el-switch>
  </el-form-item>
</template>

<script>
import i18n from "@/utils/i18n"

export default {
  name: "suggestGe-editor",
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
      if (this.optionModel.hasOwnProperty("suggestGe") && this.selectedWidget.type === "select" && this.optionModel.suggestGe) {
        if (!!this.optionModel.onFocus || this.optionModel.onFocus !== "" || !!this.optionModel.onRemoteQuery || this.optionModel.onRemoteQuery !== "") {
          this.$confirm('当前已经维护自定义脚本, 是否使用通用脚本?', '冲突提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.optionModel.onFocus = "gev3.utils.focusQuerySearchForWc(this)";
            this.optionModel.onRemoteQuery = "gev3.utils.querySearchAsyncForWc(keyword,this)";
          }).catch(() => {
          });
        } else {
          this.optionModel.onFocus = "gev3.utils.focusQuerySearchForWc(this)";
          this.optionModel.onRemoteQuery = "gev3.utils.querySearchAsyncForWc(keyword,this)";
        }
      }
    }
  }
}
</script>

<style scoped>

</style>
