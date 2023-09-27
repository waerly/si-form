<template>
  <el-form-item label=" " >
    <span slot="label">
        {{i18nt('designer.setting.rfShowRow')}}
        <el-tooltip
          effect="dark"
          content="暂不支持！"
          placement="right"
        >
          <i class="el-icon-info" ></i>
        </el-tooltip>
    </span>

    <el-radio-group v-model="optionModel.rfShowRow" class="radio-group-custom">
      <el-radio-button label="one">
        {{i18nt('designer.setting.rfRowOne')}}</el-radio-button>
      <el-radio-button label="multi">
        {{i18nt('designer.setting.rfRowMulti')}}</el-radio-button>
    </el-radio-group>
  </el-form-item>
</template>

<script>
  import i18n from "@/utils/i18n"

  export default {
    name: "rfShowRow-editor",
    mixins: [i18n],
    props: {
      designer: Object,
      selectedWidget: Object,
      optionModel: Object,
    },
    computed: {
      noLabelSetting() {
        return (this.selectedWidget.type === 'static-text') || (this.selectedWidget.type === 'html-text')
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
