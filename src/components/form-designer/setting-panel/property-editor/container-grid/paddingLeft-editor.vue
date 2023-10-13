<template>
  <div>
    <el-form-item :label="i18nt('designer.setting.paddingLeft')">
      <el-input type="number" v-model="optionModel.paddingLeft" @input.native="inputNumberHandler"
                min="0" class="hide-spin-button"></el-input>
    </el-form-item>
  </div>
</template>

<script>
  import i18n from "@/utils/i18n"
  import propertyMixin from "@/components/form-designer/setting-panel/property-editor/propertyMixin"

  export default {
    name: "paddingLeft-editor",
    mixins: [i18n, propertyMixin],
    props: {
      designer: Object,
      selectedWidget: Object,
      optionModel: Object,
    },

  }
</script>

<style scoped>

</style>
