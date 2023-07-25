<template>
  <form-item-wrapper :designer="designer" :field="field" :rules="rules" :design-state="designState"
                     :parent-widget="parentWidget" :parent-list="parentList" :index-of-parent-list="indexOfParentList"
                     :sub-form-row-index="subFormRowIndex" :sub-form-col-index="subFormColIndex" :sub-form-row-id="subFormRowId">

<!--    height 40 ,36,32,28-->
    <div v-show="false" :class="'el-input el-input--'+field.options.size">
      <div v-for="item in field.options.optionItems" :key="item.value" >
        <div v-if="fieldModel===item.value" :style="'overflow: auto;border-radius: 5px;padding: 0 4px;'+(item.bc?'background:'+item.bc:'')">
          <span v-if="item.i" :style="'float: left;margin-right:4px;'+(item.c?'color:'+item.c:'')"><i :class="item.i"/></span>
          <span v-if="item.img" :style="'float: left;margin-right:4px;'+(item.c?'color:'+item.c:'')">
            <img alt="item.label" :style="'width: auto;max-width: 60px;height: 28px;'+(field.options.selectProfile?'border-radius: 50%;width:28px;':'')" :src="item.img"/>
          </span>
          <span :style="'float: left;'+(item.c?'color:'+item.c:'')">{{ item.label }}</span>
        </div>
      </div>
    </div>

    <el-select ref="fieldEditor" v-model="fieldModel" :class="'full-width-input'"
               :disabled="field.options.disabled"
               :size="field.options.size"
               :clearable="field.options.clearable"
               :filterable="field.options.filterable"
               :allow-create="field.options.allowCreate"
               :default-first-option="allowDefaultFirstOption"
               :automatic-dropdown="field.options.automaticDropdown"
               :multiple="field.options.multiple" :multiple-limit="field.options.multipleLimit"
               :placeholder="field.options.placeholder || i18nt('render.hint.selectPlaceholder')"
               :remote="field.options.remote" :remote-method="remoteQuery"
               @focus="handleFocusCustomEvent" @blur="handleBlurCustomEvent"
               @change="handleChangeEvent">
      <template #prefix v-if="isUseCustom(getItem())&&!field.options.multiple">
        <div class="custom-selection"  >
          <div :style="'overflow: auto;border-radius: 5px;padding: 0 4px;'+(currentItem.bc?'background:'+currentItem.bc:'')">
            <span class="custom-selection-icon" v-if="currentItem.i" :style="'float: left;margin-right:4px;'+(currentItem.c?'color:'+currentItem.c:'')"><i :class="currentItem.i"/></span>
            <span class="custom-selection-img" v-if="currentItem.img" :style="'float: left;margin-right:4px;'+(currentItem.c?'color:'+currentItem.c:'')">
              <img alt="currentItem.label" :style="'width: auto;max-width: 60px;height: 34px;'+(field.options.selectProfile?'border-radius: 50%;width:34px;':'')" :src="currentItem.img"/>
            </span>
            <span class="custom-selection-title"  :style="'float: left;'+(currentItem.c?'color:'+currentItem.c:'')">{{ currentItem.label }}</span>
            <span class="custom-selection-value" v-if="field.options.selectShowValue" :style="'float: right; font-size: 12px;'+(currentItem.c?'color:'+currentItem.c:'')">{{ currentItem.value }}</span>
          </div>
        </div>

      </template>
      <el-option v-for="item in field.options.optionItems" :key="item.value" :label="item.label"
                 :value="item.value" :disabled="item.disabled">
        <div :style="'overflow: auto;border-radius: 5px;padding: 0 4px;'+(item.bc?'background:'+item.bc:'')">
          <span v-if="item.i" :style="'float: left;margin-right:4px;'+(item.c?'color:'+item.c:'')"><i :class="item.i"/></span>
          <span v-if="item.img" :style="'float: left;margin-right:4px;'+(item.c?'color:'+item.c:'')">
            <img alt="item.label" :style="'width: auto;max-width: 60px;height: 34px;'+(field.options.selectProfile?'border-radius: 50%;width:34px;':'')" :src="item.img"/>
          </span>
          <span :style="'float: left;'+(item.c?'color:'+item.c:'')">{{ item.label }}</span>
          <span v-if="field.options.selectShowValue" :style="'float: right; font-size: 12px;'+(item.c?'color:'+item.c:'')">{{ item.value }}</span>
        </div>
      </el-option>
    </el-select>
  </form-item-wrapper>
</template>

<script>
  import FormItemWrapper from './form-item-wrapper'
  import emitter from '@/utils/emitter'
  import i18n, {translate} from "@/utils/i18n";
  import fieldMixin from "@/components/form-designer/form-widget/field-widget/fieldMixin";
  import {isEmptyStr} from "@/utils/util";

  export default {
    name: "select-widget",
    componentName: 'FieldWidget',  //必须固定为FieldWidget，用于接收父级组件的broadcast事件
    mixins: [emitter, fieldMixin, i18n],
    props: {
      field: Object,
      parentWidget: Object,
      parentList: Array,
      indexOfParentList: Number,
      designer: Object,

      designState: {
        type: Boolean,
        default: false
      },

      subFormRowIndex: { /* 子表单组件行索引，从0开始计数 */
        type: Number,
        default: -1
      },
      subFormColIndex: { /* 子表单组件列索引，从0开始计数 */
        type: Number,
        default: -1
      },
      subFormRowId: { /* 子表单组件行Id，唯一id且不可变 */
        type: String,
        default: ''
      },

    },
    components: {
      FormItemWrapper,
    },
    inject: ['refList', 'formConfig', 'globalOptionData', 'globalModel'],
    data() {
      return {
        oldFieldValue: null, //field组件change之前的值
        fieldModel: null,
        rules: [],
        sizeMap : {
          large: 40,
          medium: 36,
          small: 32,
          mini: 28
        },
        currentItem:{},
      }
    },
    computed: {
      allowDefaultFirstOption() {
        return (!!this.field.options.filterable && !!this.field.options.allowCreate)
      },

    },
    beforeCreate() {
      /* 这里不能访问方法和属性！！ */
    },

    created() {
      /* 注意：子组件mounted在父组件created之后、父组件mounted之前触发，故子组件mounted需要用到的prop
         需要在父组件created中初始化！！ */
      this.initOptionItems()
      this.initFieldModel()
      this.registerToRefList()
      this.initEventHandler()
      this.buildFieldRules()

      this.handleOnCreated()

    },

    mounted() {
      this.handleOnMounted()
    },

    beforeDestroy() {
      this.unregisterFromRefList()
    },

    methods: {
      //判断当前是否实用颜色
      isUseCustom(item){
        return item && (!isEmptyStr(item.c) || !isEmptyStr(item.bc) || !isEmptyStr(item.i) || !isEmptyStr(item.img));
      },
      getItem(){
        if (this.currentItem && Object.keys(this.currentItem).length > 0 && this.currentItem.value == this.fieldModel) {
          return this.currentItem;
        }
        let currentItem=this.field.options.optionItems.find(item=>item.value==this.fieldModel);
        this.currentItem = currentItem;
        return currentItem;
      }
    }
  }
</script>

<style lang="scss" scoped>
  @import "../../../../styles/global.scss"; //* form-item-wrapper已引入，还需要重复引入吗？ *//

  .full-width-input {
    width: 100% !important;
  }
  .custom-selection {
    text-align: left;
    display: inline-block;
    overflow: hidden;
    height: calc(100% - 4px);
    background: white;
    margin-top: 3px;
    padding-left: 9px;
    padding-right: 9px;
  }
  .el-input__prefix .custom-selection .custom-selection-icon,
  .el-input__prefix .custom-selection .custom-selection-title,
  .el-input__prefix .custom-selection .custom-selection-value
  {
    line-height: 30px;
  }
  .el-input__prefix .custom-selection .custom-selection-img{
    line-height: 30px;
  }
  .el-select--mini .el-input__prefix .custom-selection .custom-selection-icon,
  .el-select--mini .el-input__prefix .custom-selection .custom-selection-title,
  .el-select--mini .el-input__prefix .custom-selection .custom-selection-value
  {
    line-height: 22px;
  }
  .el-select--mini .el-input__prefix .custom-selection .custom-selection-img{
    line-height: 22px;
  }

  .el-input__inner{

  }

</style>
