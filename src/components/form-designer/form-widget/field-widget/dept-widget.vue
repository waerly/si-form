<template>
  <form-item-wrapper :designer="designer" :field="field" :rules="rules" :design-state="designState"
                     :parent-widget="parentWidget" :parent-list="parentList" :index-of-parent-list="indexOfParentList"
                     :sub-form-row-index="subFormRowIndex" :sub-form-col-index="subFormColIndex"
                     :sub-form-row-id="subFormRowId">

    <!--    height 40 ,36,32,28-->
    <div v-show="true" :class="' el-input el-input--'+field.options.size">
      <div class="si-select-user" @click="handlerSelectUserClick">
        <!--   是否多选      -->
        <div v-if="field.options.multiple" class="si-select-user-container">
          <div v-for="item in fieldModel" :key="item.id" class="si-select-user-container-item">

            <div v-if="true"
                 :style="'overflow: auto;border-radius: 5px;display: flex;padding: 0 4px;'">
              <div v-if="isNotEmptyForAvatar(item)" class="si-select-user-profile-root">
                <span class="si-select-user-profile">
                <img alt="item.label"
                     class="si-select-user-profile-img"
                     :src="getAvatar(item)"/>
                </span>
              </div>
              <div v-else class="si-select-user-profile-root">
                <span v-if="!isObjEmpty(item)" class="si-select-user-profile"
                      :style="'background:'+ stringToHex(getFastUserName(item))">{{ getFastUserName(item) }}</span>
              </div>
              <span class="si-select-user-profile-name">{{
                  item.name
                }}</span>
            </div>
          </div>
        </div>
        <div v-else>
          <div class="si-select-user-container">
            <div v-if="!isObjEmpty(fieldModel)"
                 :style="'overflow: auto;border-radius: 5px;display: flex;padding: 0 4px;'">
              <div v-if="isNotEmptyForAvatar(fieldModel)" class="si-select-user-profile-root">
                  <span class="si-select-user-profile">
                  <img alt="item.label"
                       class="si-select-user-profile-img"
                       :src="getAvatar(fieldModel)"/>
                  </span>
              </div>
              <div v-else class="si-select-user-profile-root">
                  <span v-if="fieldModel&&!isObjEmpty(fieldModel)" class="si-select-user-profile"
                        :style="'background:'+ stringToHex(getFastUserName(fieldModel))">{{
                      getFastUserName(fieldModel)
                    }}</span>
              </div>
              <span class="si-select-user-profile-name">{{
                  fieldModel.name
                }}</span>
            </div>
          </div>
        </div>

      </div>

    </div>

    <!-- 临时测试选择框    -->
    <div>
      <!--  测试新版样式-->
      <dept-selection ref="userSelectionRef"  @user-selection-data="handleUserSelectionData"
                      :visible="userSelectionVisible" :default-checked="defaultCheckedData"
                      :type="userIsMulti?'multi':'radio'"></dept-selection>

    </div>

<!--    <el-select ref="fieldEditor" v-model="fieldModel" class="full-width-input"
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
      <el-option v-for="item in field.options.optionItems" :key="item.value" :label="item.label"
                 :value="item.value" :disabled="item.disabled">
        <div :style="'overflow: auto;border-radius: 5px;padding: 0 4px;'+(item.bc?'background:'+item.bc:'')">
          <span v-if="item.i" :style="'float: left;margin-right:4px;'+(item.c?'color:'+item.c:'')"><i :class="item.i"/></span>
          <span v-if="item.img" :style="'float: left;margin-right:4px;'+(item.c?'color:'+item.c:'')">
            <img alt="item.label"
                 :style="'width: auto;max-width: 60px;height: 34px;'+(field.options.selectProfile?'border-radius: 50%;width:34px;':'')"
                 :src="item.img"/>
          </span>
          <span :style="'float: left;'+(item.c?'color:'+item.c:'')">{{ item.label }}</span>
          <span v-if="field.options.selectShowValue"
                :style="'float: right; font-size: 12px;'+(item.c?'color:'+item.c:'')">{{ item.value }}</span>
        </div>
      </el-option>
    </el-select>-->

  </form-item-wrapper>
</template>

<script>
import FormItemWrapper from './form-item-wrapper'
import emitter from '@/utils/emitter'
import i18n, {translate} from "@/utils/i18n";
import fieldMixin from "@/components/form-designer/form-widget/field-widget/fieldMixin";
import axios from "axios";
import {arrayClear, isArray, isEmpty, isListEmpty, isObj, isObjEmpty} from "@/utils/util";
import deptSelection from "@/components/ge/dept-selection/index.vue";

export default {
  name: "dept-widget",
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
    FormItemWrapper, deptSelection
  },
  inject: ['refList', 'formConfig', 'globalOptionData', 'globalModel'],
  data() {
    return {
      oldFieldValue: null, //field组件change之前的值
      fieldModel: null,
      rules: [],
      sizeMap: {
        large: 40,
        medium: 36,
        small: 32,
        mini: 28
      },


      userIsMulti: this.field.options.multiple,//是否为多选
      defaultCheckedData: [],
      userSelectionVisible: false,
      userCheckedData:[],//当前选中数据


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
    // console.log(this.formModel,this.field)
    this.initOptionItems()
    this.initFieldModel()
    this.registerToRefList()
    this.initEventHandler()
    this.buildFieldRules()

    this.handleOnCreated()


    // this.initFieldModelForRadio()
    this.$nextTick(()=>{
      this.initCheckedData();
      // console.log("表单：",this.formDataModel)
    })
  },

  mounted() {
    this.handleOnMounted()
  },

  beforeDestroy() {
    this.unregisterFromRefList()
  },

  methods: {
    isObjEmpty,
    //初始化选择器中数据
    initCheckedData() {
      if (this.userIsMulti) {
        //尝试使用兼容性方式
        if (isObj(this.fieldModel)) {
          this.fieldModel = [this.fieldModel];
        }
        this.fieldModel.forEach(f => {
          this.defaultCheckedData.push(f.id);
        })
      } else {
        //尝试使用兼容性方式
        if (isArray(this.fieldModel)) {
          if (!isListEmpty(this.formModel)) {
            this.fieldModel = this.fieldModel[0];
          }else {
            this.fieldModel = {};
          }
        }
        this.defaultCheckedData.push(this.fieldModel.id);
      }
    },
    //根据明确取首字母
    getFastUserName(row) {
      if (row && !isEmpty(row.name)) {
        return row.name.substring(0, 1);
      }
      return "";
    },
    stringToHex(str) {
      let val = "";
      if (str.length === 1) {
        str += 'a';
      }
      for (let i = 0; i < str.length; i++) {
        if (val === "") {
          val = str.charCodeAt(i).toString(16);
        } else {
          val += "" + str.charCodeAt(i).toString(16);
        }
      }
      if (val.length === 4) {
        val += "ee";
      }
      return "#" + val;
    },
    //判断头像是否为空
    isNotEmptyForAvatar(row) {
      return row && row.avatar && !isEmpty(row.avatar);
    },
    //获取头像完整地址
    getAvatar(row) {
      if (row) {
        let avatar = row.avatar;
        if (!isEmpty(avatar)) {
          if (avatar.indexOf("http:") !== -1 || avatar.indexOf("https:") !== -1) {
            return avatar;
          } else {
            return this.getCtx() + avatar;
          }
        }
      }
      return null;
      //获取头像完整地址
      // if (!!this.fieldModel) {
      //   // let avatar=this.fieldModel
      //   console.log("模型：",this.fieldModel)
      // }
      //如果是数组时

    },
    //提前名称
    //单选时初始化格式
    initFieldModelForRadio() {
      if (!!this.fieldModel && this.fieldModel.indexOf("{") !== -1) {
        this.fieldModel = JSON.parse(this.fieldModel || "{}");
      }
    },
    //将数据转为模型对象
    checkedDataToFieldMode(data=[]){
      if (this.userIsMulti) {
        arrayClear(this.fieldModel);
        data.forEach(d=>{
          this.fieldModel.push({id:d.id,name:d.label, avatar: d.avatar})
        })
        if (isListEmpty(data)) {
          arrayClear(this.fieldModel);
        }
      }else{
        data.forEach(d=> {
          this.fieldModel = {id: d.id, name: d.label, avatar: d.avatar};
        });
        if (isListEmpty(data)) {
          this.fieldModel = {};
        }
      }

      if (this.formModel&&this.field&&this.field.options&&this.field.options.name) {
        this.$set(this.formModel, this.field.options.name, this.fieldModel);
      }
    },
    handleUserSelectionData(data) {
      this.userCheckedData = data;
      //将数据回写
      arrayClear(this.defaultCheckedData);
      data.forEach(d=>{
        this.defaultCheckedData.push(d.id);
      })
      this.checkedDataToFieldMode(data);
    },
    handlerSelectUserClick() {
      if (this.field.options.disabled) {
        return;
      }
      this.userSelectionVisible = !this.userSelectionVisible;
      this.$refs['userSelectionRef'].dialogVisible = true;
      this.$refs['userSelectionRef'].setType(this.userIsMulti ? 'multi' : 'radio');
      setTimeout(()=>{
        this.$refs['userSelectionRef'].initDefaultChecked(this.defaultCheckedData);
      },400)
    }
  },
  watch: {},
}
</script>

<style lang="scss" scoped>
@import "../../../../styles/global.scss"; //* form-item-wrapper已引入，还需要重复引入吗？ *//

.full-width-input {
  width: 100% !important;
}

::v-deep .el-table .si-uer-hidden-row {
  display: none;
}

.si-select-user ::v-deep .si-select-user-container {
  display: flex;
  justify-items: start;
  flex-flow: wrap;
}

.si-select-user .si-select-user-container ::v-deep .si-select-user-container-item {
  padding: 3px 5px;
}

::v-deep .si-select-user-profile-root {
  display: inline-block;
  margin-right: 4px;
}

::v-deep .si-select-user-profile-name {
  height: 30px;
  line-height: 30px;
}

::v-deep .si-select-user-profile {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  overflow: hidden;
  border: 1px solid #ccc;
  display: block;
  line-height: 28px;
  text-align: center;
  background: #b45daf;
  color: white;
}

::v-deep .si-select-user-profile-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.si-select-user {
  border: 1px solid #e0e0e0;
  min-height: 26px;
  padding: 4px 2px;
}

.el-tag {
  margin-right: 5px;
}


.user-header {
  border: 1px solid #e0e0e0;
  border-bottom: none;
  padding: 4px 4px 6px;
  min-height: 30px;
}

.user-aside {
  border-right: 1px solid #e0e0e0;
  padding: 0 5px;
}

.user-container {
  border: 1px solid #e0e0e0;
}

</style>
