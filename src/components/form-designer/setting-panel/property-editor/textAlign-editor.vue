<template>
  <el-form-item :label="i18nt('designer.setting.textAlign')" v-if="noLabelSetting && (selectedWidget.type !== 'button')">
    <el-select v-model="optionModel.textAlign">
      <el-option label="center" value="center"></el-option>
      <el-option label="left" value="left"></el-option>
      <el-option label="right" value="right"></el-option>
    </el-select>
  </el-form-item>
</template>

<script>
  import i18n from "@/utils/i18n"

  export default {
    name: "textAlign-editor",
    mixins: [i18n],
    props: {
      designer: Object,
      selectedWidget: Object,
      optionModel: Object,
    },
    computed: {
      noLabelSetting() {
        return (this.selectedWidget.type === 'static-text') || (this.selectedWidget.type === 'html-text')
        //|| (this.selectedWidget.type === 'divider')
      },

    }
  }
</script>

<style lang="scss" scoped>
  .radio-group-custom {
    ::v-deep .el-radio-button__inner {
      padding-left: 12px;
      padding-right: 12px;
    }
  }
</style>
