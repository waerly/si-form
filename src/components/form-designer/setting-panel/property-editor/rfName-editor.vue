<template>
  <div>
  <el-divider class="custom-divider-margin-top">{{i18nt('designer.setting.rfName')}}</el-divider>
  <el-form-item :label="i18nt('designer.setting.rfName')">
    <el-select v-model="optionModel.rfName" filterable clearable>
      <el-option v-for="item in subFunItems" :key="item.value" :label="item.label"
                 :value="item.value">
      </el-option>
    </el-select>
  </el-form-item>
  </div>
</template>

<script>
  import i18n from "@/utils/i18n"

  export default {
    name: "rfName-editor",
    mixins: [i18n],
    props: {
      designer: Object,
      selectedWidget: Object,
      optionModel: Object,
    },
    data() {
      return {
        siGeFunReq:{},
        siGeFun:{},
        subFunItems: [],
      }
    },
    created() {
      this.siGeFunReq = this.designer.vueInstance.siGeFunReq;
      this.siGeFun = this.designer.vueInstance.siGeFunReq.siGeFun;
      this.parseSetReqToSunFunOptionItems(this.siGeFunReq);
    },
    methods:{
      parseSetReqToSunFunOptionItems(siGeFunReq){

        siGeFunReq.subSiGeFunList.forEach(subFun=>{
          let sf = {};
          sf.value = subFun.funCode;
          sf.label = subFun.functionName;
          this.subFunItems.push(sf);
        })
      }
    }
  }
</script>

<style scoped>

</style>
