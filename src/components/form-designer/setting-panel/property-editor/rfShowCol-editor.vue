<template>
  <el-form-item v-if="optionModel.rfCustomCol" class="si-form-show-col" label-width="0">
<!--    <el-divider class="custom-divider-margin-top">{{i18nt('designer.setting.optionsSetting')}}</el-divider>-->
<!--    <el-divider class="custom-divider-margin-top">{{i18nt('designer.setting.rfShowCol')}}</el-divider>-->


    <el-form-item label=" ">
      <span slot="label">
        {{i18nt('designer.setting.rfShowCol')}}

        <el-tooltip
          effect="dark"
          content="刷新列表"
          placement="right"
        >
          <i class="el-icon-refresh" @click="onloadFunCol"></i>
        </el-tooltip>
      </span>
      <el-select value="" clearable multiple filterable @change="handlerChangeShowCol">
        <el-option v-for="item in optionItems" :key="item.value" :label="item.label"
                   :value="item.value">
          <template #default><span :class="checkOptionItemsMap.hasOwnProperty(item.value)?'item-option-selected':''">{{item.label}}</span></template>

        </el-option>
      </el-select>
    </el-form-item>

    <div class="option-items-pane">
      <el-radio-group v-if="(selectedWidgetData.type === 'relation-fun')"
                       @change="emitDefaultValueChange">
        <draggable tag="ul" :list="checkOptionItems"
                   v-bind="{group:'optionsGroup', ghostClass: 'ghost', handle: '.drag-option'}">
          <li v-for="(option, idx) in checkOptionItems" :key="idx">
            <el-radio :label="option.value">
              <el-input v-if="false" v-model="option.value" size="mini" style="width: 80px"></el-input>
              <el-input readonly v-model="option.label" size="mini" style="width: 190px">
<!--                <template #prefix>
                  <i @click="handlerIconClick(option)" :style="{padding:'3px','border-radius':'3px', 'color':option.c,'background-color':option.bc}" :class="option.i?option.i:'el-icon-edit'">&nbsp;</i>
                </template>-->
              </el-input>
              <i class="iconfont icon-drag drag-option"></i>
              <el-button circle plain size="mini" type="danger" @click="deleteOption(option, idx)"
                         icon="el-icon-minus" class="col-delete-button"></el-button>
            </el-radio>
          </li>
        </draggable>
      </el-radio-group>
    </div>

  </el-form-item>
</template>

<script>
  import i18n from "@/utils/i18n"
  import emitter from '@/utils/emitter'
  import OptionItemsSetting from "@/components/form-designer/setting-panel/option-items-setting"
  import Draggable from 'vuedraggable'
  import fieldMixin from "@/components/form-designer/form-widget/field-widget/fieldMixin";
  import axios from "axios";
  import fa from "element-ui/src/locale/lang/fa";

  export default {
    name: "rfShowCol-editor",
    mixins: [emitter,i18n,fieldMixin],
    props: {
      designer: Object,
      selectedWidget: Object,
      optionModel: Object,
    },
    components: {
      Draggable,
      OptionItemsSetting,
    },
    inject: ['refList', 'formConfig', 'globalOptionData', 'globalModel'],
    data() {
      return {
        siGeFunReq:{},
        siGeFun:{},
        optionItems:[],
        checkOptionItems:[],
        checkOptionItemsMap:{},
        selectedWidgetData: this.selectedWidget,
        optionModelData: this.optionModel,
      }
    },
    created() {
      this.siGeFunReq = this.designer.vueInstance.siGeFunReq;
      this.siGeFun = this.designer.vueInstance.siGeFunReq.siGeFun;
      this.onloadOptionsValue();
      // this.parseSetReqToOptionItems(this.siGeFun);
      this.onloadFunCol(true);
      //监听字段组件选中事件
      this.designer.handleEvent('field-selected', (parentWidget) => {
        if (this.designer.selectedWidget.type === 'relation-fun') {
          this.selectedWidgetData = this.designer.selectedWidget;
          // console.log("this,",this.selectedWidgetData)
          this.optionModelData = this.designer.selectedWidget.options;
          this.onloadOptionsValue();
        }
      })
    },
    destroyed() {
      //存在监听未撤销的问题
      // this.designer.removeEvent("field-selected")
    },
    methods: {

      //加载功能列
      onloadFunCol(firstLoad = false) {
        let self = this;
        let funCode = this.optionModelData.rfName;
        if (this.optionModelData.rfCustomFunCode != null && this.optionModelData.rfCustomFunCode !== "") {
          funCode = this.optionModelData.rfCustomFunCode;
        }
        if (!funCode || funCode === "") {
          this.$message.warning("请先选择功能或者输入功能编码!")
          return;
        }
        axios.post(`${this.getCtx()}core/find_ge_fun_info/${funCode}`).then(res => {
          if (res.status === 200 && res.data) {
            let resData = res.data;
            if (!resData.data) {
              this.$message.warning("没有找到对应功能，请核实！")
              return
            }
            if (resData.code === 0) {
              let data = resData.data;
              self.parseSetReqToOptionItems(data.siGeFun)
              if (!firstLoad) {
                this.checkOptionItems.splice(0, this.checkOptionItems.length);
              }
            }
          } else {
            this.$message.warning("异常:", res.status, res.statusText)
          }
        }).catch(res => {
        });
      },
      refreshHandler(){
        console.log("刷新")
      },
      //解析组件中的值
      onloadOptionsValue(){
        this.checkOptionItems.splice(0, this.checkOptionItems.length);
        if (this.optionModelData.rfShowCol && this.optionModelData.rfShowCol.length > 0) {
          this.optionModelData.rfShowCol.forEach(col=>{
            this.checkOptionItems.push(col);
          })
        }
      },

      //解析req中的列，并将其转为 value 和 label的格式
      parseSetReqToOptionItems(siGeFun){
        let columns=siGeFun.columns;
        let items=[];
        columns.forEach(col=>{
          let item = {};
          if (col.inReserved === "1") {
            return;
          }
          item.value = col.javaField;
          item.label = col.columnComment;
          items.push(item);
        })
        this.optionItems = items;
        this.checkOptionItemsToMap();
      },
      emitDefaultValueChange() {
        //选择默认改为刷新
        //选择默认的时候
        // if (!!this.designer && !!this.designer.formWidget) {
        //   let fieldWidget = this.designer.formWidget.getWidgetRef(this.selectedWidget.options.name)
        //   if (!!fieldWidget && !!fieldWidget.refreshDefaultValue) {
        //     fieldWidget.refreshDefaultValue()
        //   }
        // }
      },
      deleteOption(option, index) {
        this.checkOptionItems.splice(index, 1)
        this.checkOptionItemsToMap();
        this.optionModelData.rfShowCol = this.checkOptionItems;
      },
      checkOptionItemsToMap(){
        this.checkOptionItems.forEach((opt,index)=>{
          // opt.index=index;
          this.checkOptionItemsMap[opt.value] = opt;
        })
      },
      handlerChangeShowCol(val){

        val.forEach(v=>{
          //删除
          if (this.checkOptionItemsMap.hasOwnProperty(v)) {
            let itemIndex = -1;
            this.checkOptionItems.forEach((item,index)=>{
              if (item.value === v) {
                itemIndex = index;
              }
            })
            if (itemIndex >= 0) {
              this.checkOptionItems.splice(itemIndex, 1);
            }
            delete this.checkOptionItemsMap[v];
          }else {
            let opt = this.optionItems.find(opt => opt.value === v);
            // opt.index = this.checkOptionItems.length;
            this.checkOptionItems.push(opt);
            this.checkOptionItemsMap[v] = opt;
          }
        })

        //设置当前值
        this.optionModelData.rfShowCol = this.checkOptionItems;

      },

      handlerIconClick(opt) {
        console.log("jihuo>>>>")
        // this.optionStyleEditorFlag = !this.optionStyleEditorFlag;

        // this.optionStyleEditorFlag = true;
        // this.currentOption = opt;

      },
    }
  };
</script>

<style scoped>
.si-form-show-col{

}
.item-option-selected{
  color: #aaaaaa;
}

.option-items-pane ul {
  padding-inline-start: 6px;
  padding-left: 6px;  /* 重置IE11默认样式 */
}

li.ghost{
  background: #fff;
  border: 2px dotted $--color-primary;
}

.drag-option {
  cursor: move;
}

.small-padding-dialog ::v-deep .el-dialog__body {
  padding: 10px 15px;
}

.dialog-footer .el-button {
  width: 100px;
}
</style>
