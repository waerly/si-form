<template>
  <el-form-item :label="i18nt('designer.setting.activeVal')">
    <el-input v-model="optionModel.activeVal"></el-input>
  </el-form-item>
</template>

<script>
  import i18n from "@/utils/i18n"

  export default {
    name: "activeVal-editor",
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
