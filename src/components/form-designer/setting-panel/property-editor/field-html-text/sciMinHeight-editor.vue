<template>
  <el-form-item :label="i18nt('designer.setting.sciMinHeight')">
    <el-input v-model="optionModel.sciMinHeight"></el-input>
  </el-form-item>
</template>

<script>
  import i18n from "@/utils/i18n"

  export default {
    name: "sciMinHeight-editor",
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
