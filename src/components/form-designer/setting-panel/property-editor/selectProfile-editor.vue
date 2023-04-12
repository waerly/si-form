<!--图像以头像显示-->
<template>
  <el-form-item :label="i18nt('designer.setting.selectProfile')">
    <el-switch v-model="optionModel.selectProfile"></el-switch>
  </el-form-item>
</template>

<script>
  import i18n from "@/utils/i18n"

  export default {
    name: "selectProfile-editor",
    mixins: [i18n],
    props: {
      designer: Object,
      selectedWidget: Object,
      optionModel: Object,
    },
  }
</script>

<style scoped>

</style>
