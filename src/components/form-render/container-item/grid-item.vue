<template>
  <container-item-wrapper :widget="widget" :si-ge-fun-req="siGeFunReq">

    <el-row :key="widget.id" :gutter="widget.options.gutter" class="grid-container"
            :class="[customClass]"
            :ref="widget.id" v-show="!widget.options.hidden" :style="paddingStyle">
      <template v-for="(colWidget, colIdx) in widget.cols">
        <grid-col-item :widget="colWidget" :key="colIdx" :parent-list="widget.cols"
                       :index-of-parent-list="colIdx" :parent-widget="widget"
                       :col-height="widget.options.colHeight">
          <!-- 递归传递插槽！！！ -->
          <template v-for="slot in Object.keys($scopedSlots)" v-slot:[slot]="scope">
            <slot :name="slot" v-bind="scope"/>
          </template>
        </grid-col-item>
      </template>
    </el-row>

  </container-item-wrapper>
</template>

<script>
  import emitter from '@/utils/emitter'
  import i18n from "../../../utils/i18n"
  import refMixin from "../../../components/form-render/refMixin"
  import ContainerItemWrapper from './container-item-wrapper'
  import GridColItem from './grid-col-item'
  import containerItemMixin from "./containerItemMixin"

  export default {
    name: "grid-item",
    componentName: 'ContainerItem',
    mixins: [emitter, i18n, refMixin, containerItemMixin],
    components: {
      ContainerItemWrapper,
      GridColItem,
    },
    props: {
      widget: Object,
      /*通用功能*/
      siGeFunReq: {
        type: Object,
        default: () => {
          return {}
        }
      },
    },
    inject: ['refList', 'sfRefList', 'globalModel'],
    computed: {
      //填充样式
      paddingStyle(){
        let paddingTop= this.widget.options.paddingTop || ''
        let paddingRight= this.widget.options.paddingRight || ''
        let paddingBottom= this.widget.options.paddingBottom || ''
        let paddingLeft= this.widget.options.paddingLeft || ''
        let paddingStyle = "";
        if (paddingTop !== '') {
          paddingStyle += `padding-top:${paddingTop}px !important;`;
        }
        if (paddingRight !== '') {
          paddingStyle += `padding-right:${paddingRight}px;`;
        }
        if (paddingBottom !== '') {
          paddingStyle += `padding-bottom:${paddingBottom}px;`;
        }
        if (paddingLeft !== '') {
          paddingStyle += `padding-left:${paddingLeft}px;`;
        }
        return paddingStyle;
      }
    },
    created() {
      this.initRefList()
    },
    mounted() {

    },
    beforeDestroy() {
      this.unregisterFromRefList()
    },
    methods: {

    },
  }
</script>

<style lang="scss" scoped>

</style>
