<template>
  <form-item-wrapper :designer="designer" :field="field" :rules="rules" :design-state="designState"
                     :parent-widget="parentWidget" :parent-list="parentList" :index-of-parent-list="indexOfParentList"
                     :sub-form-row-index="subFormRowIndex" :sub-form-col-index="subFormColIndex"
                     :sub-form-row-id="subFormRowId">
    <!--    <vue-editor  v-model="fieldModel" :editor-toolbar="customToolbar"
                    :disabled="field.options.disabled" :placeholder="field.options.placeholder"
                    @text-change="handleRichEditorChangeEvent"
                    @focus="handleRichEditorFocusEvent" @blur="handleRichEditorBlurEvent">
        </vue-editor>-->
    <div style="border: 1px solid #ccc;">
      <Toolbar
          style="border-bottom: 1px solid #ccc"
          :editor="editor"
          :defaultConfig="toolbarConfig"
          :mode="mode"
      />
      <Editor ref="fieldEditor"
              style="min-height: 200px; overflow-y: hidden;"
              v-model="fieldModel"
              :defaultConfig="editorConfig"
              :mode="mode" @onFocus="handleRichEditorFocusEvent" @onBlur="handleRichEditorBlurEvent"
              @onCreated="onCreated"
      />
    </div>
  </form-item-wrapper>
</template>

<script>
import FormItemWrapper from './form-item-wrapper'
import {VueEditor} from 'vue2-editor'
import {Editor, Toolbar} from '@wangeditor/editor-for-vue'
import emitter from '@/utils/emitter'
import i18n, {translate} from "@/utils/i18n"
import {deepClone, generateId} from "@/utils/util"
import fieldMixin from "@/components/form-designer/form-widget/field-widget/fieldMixin"

export default {
  name: "rich2-editor-widget",
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
    VueEditor, Editor, Toolbar
    // VueEditor: resolve => { //懒加载！！
    //   require(['vue2-editor'], ({VueEditor}) => resolve(VueEditor))
    // }
  },
  inject: ['refList', 'formConfig', 'globalOptionData', 'globalModel'],
  data() {
    return {
      oldFieldValue: null, //field组件change之前的值
      fieldModel: null,
      rules: [],
      customToolbar: [], //富文本编辑器自定义工具栏
      valueChangedFlag: false, //vue2-editor数据值是否改变标志
      //wang
      mode: 'default',
      editor: null,
      // html: '<p>hello</p>',
      toolbarConfig: {},//富文本配置
      editorConfig: {placeholder: '请输入内容...'},
    }
  },
  computed: {},
  watch: {
    'field.options.disabled': {
      handler(oldVal, newVal) {
        this.changeDisabled();
      }
    },
    fieldModel: {
      handler() {
        this.handleRichEditorChangeEvent();
      }
    }
  },
  beforeCreate() {
    /* 这里不能访问方法和属性！！ */
  },

  created() {
    /* 注意：子组件mounted在父组件created之后、父组件mounted之前触发，故子组件mounted需要用到的prop
       需要在父组件created中初始化！！ */
    this.initFieldModel()
    this.registerToRefList()
    this.initEventHandler()
    this.buildFieldRules()

    this.handleOnCreated()
    this.setUploadImgConfig();
    this.setUploadVideoConfig();
  },

  mounted() {
    this.handleOnMounted()
  },

  beforeDestroy() {
    //销毁编辑器
    const editor = this.editor
    if (editor == null) return
    editor.destroy() // 组件销毁时，及时销毁编辑器
    this.unregisterFromRefList()
  },
  methods: {
    //设置配置信息
    setUploadImgConfig() {
      let geUrl = null;
      if (this.field.options.uploadGe) {
        let ctxPrefix = "";
        try {
          // noinspection JSUnresolvedVariable
          ctxPrefix = ctx;
          // eslint-disable-next-line no-empty
        } catch (e) {
        }
        geUrl = ctxPrefix + "/common/upload";
      }

      this.editorConfig.MENU_CONF = {};
      this.editorConfig.MENU_CONF['uploadImage'] = {
        server: !!this.field.options.uploadURL && this.field.options.uploadURL !== '' ? this.field.options.uploadURL : this.field.options.uploadGe ? geUrl : this.field.options.uploadURL,//测试服务器
        fieldName: 'file_data',
        meta: {token: 'xxx', fileId: generateId()},
        metaWithUrl: true,
        headers: {Accept: '*/*'},
        maxFileSize: this.field.options.fileMaxSize * 1024 * 1024, // 10M
        // base64LimitSize: 5 * 1024,
        customInsert(res, insertFn) {
          const imgInfo = {}
          imgInfo.url = res.url;
          imgInfo.alt = res.fileName;
          imgInfo.href = res.url;
          const {url, alt, href} = imgInfo
          if (!url) throw new Error(`Image url is empty`)
          insertFn(url, alt, href)
        },
      }
      this.editorConfig.MENU_CONF['uploadVideo'] = {
        server: !!this.field.options.uploadURL && this.field.options.uploadURL !== '' ? this.field.options.uploadURL : this.field.options.uploadGe ? geUrl : this.field.options.uploadURL,//测试服务器
        fieldName: 'file_data',
        meta: {token: 'xxx', fileId: generateId()},
        metaWithUrl: true,
        headers: {Accept: '*/*'},
        maxFileSize: this.field.options.fileMaxSize * 1024 * 1024, // 10M
        customInsert(res, insertFn) {
          const imgInfo = {}
          imgInfo.url = res.url;
          imgInfo.alt = res.fileName;
          imgInfo.poster = res.cover;
          const {url, poster} = imgInfo
          if (!url) throw new Error(`Video url is empty`)
          insertFn(url, poster)
        },
      }
      // console.log("地址", this.editorConfig.MENU_CONF['uploadImage'].server)
    },
    setUploadVideoConfig() {

    },
    onCreated(editor) {
      this.editor = Object.seal(editor) // 一定要用 Object.seal() ，否则会报错
      this.initCreated();
    },
    initCreated() {
      // let editContainer=this.$refs.fieldEditor.$el.querySelector(".w-e-text-container");//.addEventListener()
      // let editor=editContainer.querySelector("div[data-slate-editor]");
      // editor.addEventListener("focus",function(){
      //   this.handleRichEditorFocusEvent();
      // })
      // editor.addEventListener("blur",function(){
      //   this.handleRichEditorBlurEvent();
      // })
      this.changeDisabled();
    },
    changeDisabled() {
      if (this.field.options.disabled) {
        this.editor.disable();
      } else {
        this.editor.enable();
      }
    },
    //wang end
    handleRichEditorChangeEvent() {
      this.valueChangedFlag = true
      this.syncUpdateFormModel(this.fieldModel)
    },
    handleRichEditorFocusEvent() {
      this.oldFieldValue = deepClone(this.fieldModel)
    },
    handleRichEditorBlurEvent() {
      if (this.valueChangedFlag) {
        this.emitFieldDataChange(this.fieldModel, this.oldFieldValue)
        this.valueChangedFlag = false
      }
    },

  }
}
</script>

<style src="@wangeditor/editor/dist/css/style.css"></style>

<style lang="scss" scoped>
@import "../../../../styles/global.scss"; //* form-item-wrapper已引入，还需要重复引入吗？ *//

.full-width-input {
  width: 100% !important;
}

</style>
