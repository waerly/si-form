<!--样式编辑-->
<template>
  <div>
    <el-form-item label="名称">
      <el-input v-model="dOption.label" size="mini" style="width: 200px">
      </el-input>
    </el-form-item>
    <el-form-item label="值">
      <el-input v-model="dOption.value" size="mini" style="width: 200px"></el-input>
    </el-form-item>
    <el-form-item label="图标">
      <el-input v-model="dOption.i" size="mini" style="width: 200px">
      </el-input>
    </el-form-item>
    <el-form-item label="图片">
      <el-input v-model="dOption.img" size="mini" style="width: 200px">
      </el-input>
    </el-form-item>
    <el-form-item label="颜色">
      <div class="si-el-form-item-inline">
        <el-color-picker v-model="dOption.c" style="margin-right: 6px;"></el-color-picker>
        <el-link icon="el-icon-edit" @click="handlerAutoBackgroundColor" style="margin-right: 6px;">
          根据颜色自动取背景色
        </el-link>
        <span>快速设置：</span>
        <el-link @click="handlerAutoColor('#888888')" style="color: #888888">默认&nbsp;&nbsp;</el-link>
        <el-link @click="handlerAutoColor('#439de4')" style="color: #439de4">主要&nbsp;&nbsp;</el-link>
        <el-link @click="handlerAutoColor('#13ce66')" style="color: #13ce66">成功&nbsp;&nbsp;</el-link>
        <el-link @click="handlerAutoColor('#c2c2c2')" style="color: #c2c2c2">信息&nbsp;&nbsp;</el-link>
        <el-link @click="handlerAutoColor('#f8ac59')" style="color: #f8ac59">警告&nbsp;&nbsp;</el-link>
        <el-link @click="handlerAutoColor('#ed5565')" style="color: #ed5565">危险&nbsp;&nbsp;</el-link>
      </div>
    </el-form-item>
    <el-form-item label="背景颜色">
      <div class="si-el-form-item-inline">
        <el-color-picker v-model="dOption.bc" style="margin-right: 6px;"></el-color-picker>
      </div>

    </el-form-item>

    <el-form-item label="效果预览">
      <div>
        <span :style="{padding:'5px','border-radius':'3px', 'background-color':dOption.bc}">
          <i :style="{color:dOption.c}" :class="dOption.i?dOption.i:'el-icon-edit'">&nbsp;</i>
          {{ dOption.label }}
        </span>
      </div>
    </el-form-item>


    <el-form-item>
      <!--      <el-button type="primary" @click="handlerSaveClick">保存</el-button>-->
      <el-button type="danger" @click="handlerCleanClick">清空样式</el-button>
      <el-button type="info" @click="handlerCloseClick">取消</el-button>
    </el-form-item>
  </div>
</template>

<script>
import i18n from "@/utils/i18n"

export default {
  name: "option-style-editor",
  mixins: [i18n],
  props: {
    designer: Object,
    selectedWidget: Object,
    optionModel: Object,
    option: {
      value: "",
      label: "",
      i: "",//icon
      c: "",//color
      bc: "",//backgroundColor
    }
  },
  data() {
    return {
      dOption: this.option,
      widgetSizes: [
        {label: 'default', value: ''},
        {label: 'large', value: 'large'},
        {label: 'medium', value: 'medium'},
        {label: 'small', value: 'small'},
        {label: 'mini', value: 'mini'},
      ],
    }
  },
  methods: {
    handlerSaveClick() {
      //保存
      this.$emit('save', this.dOption);
    },
    handlerAutoColor(color) {
      this.dOption.c = color;
      this.handlerAutoBackgroundColor();
    },
    handlerAutoBackgroundColor() {
      try {
        if (this.dOption.c && this.dOption.c !== "") {
          let lightColor = $.common.getLightColor(this.dOption.c, 0.91);
          this.dOption.bc = lightColor;
        }
      } catch (e) {
        console.error(e)
      }
    },
    handlerCleanClick() {
      this.dOption.i = ""
      this.dOption.c = ""
      this.dOption.bc = ""
    },
    handlerCloseClick() {
      this.$emit('close', this);
    }
  }
}
</script>

<style scoped>

.si-el-form-item-inline {
  display: inline-flex;
}
</style>
