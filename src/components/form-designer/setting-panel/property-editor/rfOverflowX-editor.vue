<!--关联功能-->
<!--自定义关联功能编码-->
<template>
  <el-form-item :label="i18nt('designer.setting.rfOverflowX')">
<!--    <el-input @change="handlerChange" v-model="optionModel.rfHeight"></el-input>-->
    <el-select v-model="optionModel.rfOverflowX" filterable clearable allow-create default-first-option>
      <el-option v-for="(fv, fvIdx) in fieldOverflow"
                 :key="fvIdx"
                 :label="fv.label"
                 :value="fv.value">
      </el-option>
    </el-select>
  </el-form-item>
</template>

<script>
import i18n from "@/utils/i18n"

export default {
  name: "rfOverflowX-editor",
  mixins: [i18n],
  props: {
    designer: Object,
    selectedWidget: Object,
    optionModel: Object,
  },
  data() {
    return {
      fieldOverflow: [
        {value: 'scroll', label: this.i18nt('designer.cssOverflowAttr.scroll')},
        {value: 'auto', label: this.i18nt('designer.cssOverflowAttr.auto')},
        {value: 'visible', label: this.i18nt('designer.cssOverflowAttr.visible')},
        {value: 'hidden', label: this.i18nt('designer.cssOverflowAttr.hidden')},
        {value: 'clip', label: this.i18nt('designer.cssOverflowAttr.clip')},
        {value: 'overlay', label: this.i18nt('designer.cssOverflowAttr.overlay')},
      ],
    }
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
