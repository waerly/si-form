<template>
  <el-form-item v-if="optionModel.rfCustomCol" class="si-form-show-col" label-width="0">
<!--    <el-divider class="custom-divider-margin-top">{{i18nt('designer.setting.optionsSetting')}}</el-divider>-->
<!--    <el-divider class="custom-divider-margin-top">{{i18nt('designer.setting.rfShowCol')}}</el-divider>-->


    <el-form-item :label="i18nt('designer.setting.rfShowCol')">
      <el-select value="" clearable multiple filterable @change="handlerChangeShowCol">
        <el-option v-for="item in optionItems" :key="item.value" :label="item.label"
                   :value="item.value">
          <template #default><span :class="checkOptionItemsMap.hasOwnProperty(item.value)?'item-option-selected':''">{{item.label}}</span></template>
        </el-option>
      </el-select>
    </el-form-item>

    <div class="option-items-pane">
      <el-radio-group v-if="(selectedWidget.type === 'relation-fun')"
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
  import OptionItemsSetting from "@/components/form-designer/setting-panel/option-items-setting"
  import Draggable from 'vuedraggable'

  export default {
    name: "rfShowCol-editor",
    mixins: [i18n],
    props: {
      designer: Object,
      selectedWidget: Object,
      optionModel: Object,
    },
    components: {
      Draggable,
      OptionItemsSetting,
    },
    data() {
      return {
        siGeFunReq:{},
        siGeFun:{},
        optionItems:[],
        checkOptionItems:[],
        checkOptionItemsMap:{},
      }
    },
    created() {
      this.siGeFunReq = this.designer.vueInstance.siGeFunReq;
      this.siGeFun = this.designer.vueInstance.siGeFunReq.siGeFun;
      this.parseSetReqToOptionItems(this.siGeFun);
    },
    methods: {
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
      },
      checkOptionItemsToMap(){
        this.checkOptionItems.forEach((opt,index)=>{
          // opt.index=index;
          this.checkOptionItemsMap[opt.value] = opt;
        })
      },
      handlerChangeShowCol(val){
        console.log("选择",val)

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
        this.optionModel.rfShowCol = this.checkOptionItems;

      },

      handlerIconClick(opt) {
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
