<template>

  <el-dialog
    class="si-user-selection-root"
    title="人员选择"
    :visible.sync="dialogVisible"
    width="760px"
    :before-close="handleClose" append-to-body>

    <span slot="footer" class="dialog-footer">
      <el-button @click="dialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="handleClose('yes')">确 定</el-button>
    </span>

    <div>
      <!--  左侧组织机构     -->
      <el-container class="si-user-selection">
        <el-header height="auto" class="user-header">
          <div>
            <el-tag
              v-for="tag in multipleSelection"
              :key="tag.userId"
              closable
              @close="handlerTagClick(tag)"
              :type="randomTagsType()">
              {{ tag.userName }}
            </el-tag>
          </div>
        </el-header>
        <el-container class="user-container">
          <el-aside width="50%" class="user-aside">
            <div style="padding: 4px 6px">
              <template>
                <el-tabs v-model="activeName" @tab-click="handleTabClick">
                  <el-tab-pane label="组织机构" name="fraOrg">
                    <el-input
                      placeholder="输入关键字进行过滤"
                      clearable
                      v-model="filterFraText">
                      <el-button slot="append" @click="handlerFraTreeExpand">{{
                          !fraTreeExpand ? '树展开一级' : '折叠'
                        }}
                      </el-button>
                    </el-input>
                    <el-tree
                      class="filter-tree si-user-selection-left-tree"
                      :data="fraOrgData"
                      :props="defaultProps"
                      :expand-on-click-node="false"
                      @node-click="handlerTreeNodeClick"
                      :default-expand-all="fraTreeExpand"
                      :filter-node-method="filterNode"
                      ref="fraOrgTreeNode">
                      <span class="custom-tree-node" slot-scope="{ node, data }">
                        <span>{{ node.label }}</span>
                        <!--                        <span>{{data}}</span>-->
                        <span
                          style="font-size: 12px;color: #979595;"> {{
                            data.type == "0" ? '组织' : data.type == "1" ? '部门' : data.type == "2" ? '岗位' : ""
                          }}
                        </span>
                      </span>
                    </el-tree>
                  </el-tab-pane>
                  <el-tab-pane label="用户" name="user">

                    <el-input
                      clearable
                      placeholder="输入关键字进行过滤"
                      v-model="filterText">
                    </el-input>

                    <!--  :row-class-name="rowClassName"                  -->
                    <el-table
                      class="si-user-selection-left-table"
                      :size="selectionSize"
                      :ref="tableRefs.onlyUserTable"
                      :data="tableData"
                      tooltip-effect="dark"
                      row-key="userId"
                      style="width: 100%"
                      :highlight-current-row="dType==='radio'"
                      @selection-change="handleSelectionChange"
                      @current-change="handleCurrentChange"
                    >
                      <el-table-column
                        :type="getTypeForColumn()"
                        width="55">
                      </el-table-column>
                      <el-table-column
                        prop="userName"
                        label="姓名"
                        width="120">
                      </el-table-column>
                      <el-table-column
                        prop="dept.deptName"
                        label="部门"
                        show-overflow-tooltip>
                      </el-table-column>
                    </el-table>
                  </el-tab-pane>
                  <el-tab-pane label="当前用户" name="currentUser">

                    <el-input
                      clearable
                      placeholder="输入关键字进行过滤"
                      v-model="filterText">
                    </el-input>
                    <el-table
                      class="si-user-selection-left-table"
                      :size="selectionSize"
                      :ref="tableRefs.currentUserTable"
                      :data="currentUserTableData"
                      tooltip-effect="dark"
                      row-key="userId"
                      style="width: 100%"
                      :highlight-current-row="dType==='radio'"
                      @selection-change="handleSelectionChangeForCurrentUser"
                      @current-change="handleCurrentChangeForCurrentUser"
                      @row-click="handleCurrentChangeForCurrentUser"
                    >
                      <el-table-column
                        :type="getTypeForColumn()"
                        width="55">
                      </el-table-column>
                      <el-table-column
                        prop="userName"
                        label="姓名"
                        width="120">
                      </el-table-column>
                      <el-table-column
                        prop="dept.deptName"
                        label="部门"
                        show-overflow-tooltip>
                      </el-table-column>
                    </el-table>
                  </el-tab-pane>
                </el-tabs>
              </template>

            </div>
          </el-aside>
          <el-main>
            <div v-show="activeName==='fraOrg'">
              <el-table
                class="si-user-selection-main-table"
                :size="selectionSize"
                :ref="tableRefs.fraUserTable"
                :data="fraOrgTableData"
                tooltip-effect="dark"
                row-key="userId"
                style="width: 100%"
                :highlight-current-row="dType==='radio'"
                @selection-change="handleSelectionChangeForFra"
                @current-change="handleCurrentChangeForFra"
              >
                <el-table-column
                  :type="getTypeForColumn()"
                  width="55">
                </el-table-column>
                <el-table-column
                  prop="userName"
                  label="姓名"
                  width="120">
                </el-table-column>
                <el-table-column
                  prop="dept.deptName"
                  label="部门"
                  show-overflow-tooltip>
                </el-table-column>
              </el-table>
            </div>
          </el-main>
        </el-container>
      </el-container>
    </div>

  </el-dialog>


</template>

<script>
import emitter from '@/utils/emitter'
import i18n from "@/utils/i18n";
import fieldMixin from "@/components/form-designer/form-widget/field-widget/fieldMixin";
import axios from "axios";
import {arrayClear, isListEmpty} from "@/utils/util";

export default {
  name: "user-selection",
  componentName: 'FieldWidget',  //必须固定为FieldWidget，用于接收父级组件的broadcast事件
  mixins: [emitter, fieldMixin, i18n],
  props: {
    field: {
      type: Object,
      default() {
        return {}
      }
    },
    //导入数据
    selectionSize: {
      type: String,
      default() {
        return 'mini'
      }
    },
    maxBodyHeight: {
      type: String,
      default() {
        return "400px";
      }
    },
    type: {
      type: String,
      default() {
        return 'multi' //radio/multi
      }
    },
    defaultChecked: [],
    visible: {
      type: Boolean,
      default() {
        return false
      }
    }

  },
  components: {},
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

      //页签
      activeName: 'fraOrg',
      fraTreeExpand: false,

      tableRefs: {
        fraUserTable: "fraUserTable",
        onlyUserTable: "onlyUserTable",
        currentUserTable: "currentUserTable",
      },
      //用户选择
      tagsTypeMap: {default: "default", success: "success", info: "info", warning: "warning", danger: "danger"},
      tagsTypes: ["default", "success", "info", "warning", "danger"],
      tags: [
        {name: '标签一', type: ''},
        {name: '标签二', type: 'success'},
        {name: '标签三', type: 'info'},
        {name: '标签四', type: 'warning'},
        {name: '标签五', type: 'danger'}
      ],
      filterFraText: '',
      filterText: '',
      fraOrgData: [{
        id: 1,
        label: '思年华公司',
        type: "0",
        children: [{
          id: 4,
          label: '销售部门',
          type: '1',
        }]
      }],
      defaultProps: {
        children: 'children',
        label: 'label'
      },
      filterTableData: [],
      tableData: [{
        userId: '12323',
        userName: '王小虎',
        dept: {deptName: '人事部'}
      }, {
        userId: '12324',
        userName: '赵一强',
        dept: {deptName: '销售部'}
      }
      ],
      tableDataMap: {},
      currentUserTableData: [],
      currentUserTableDataMap: {},
      fraOrgTableData: [],
      fraOrgTableDataMap: {},//每次切换组织架构用户数据
      multipleSelection: [],
      multipleSelectionMap: {},//userId,userName,dept
      currentRow: null,//当前单选
      contentClassName: {
        "si-user-selection-left-tree": "si-user-selection-left-tree",
        "si-user-selection-left-table": "si-user-selection-left-table",
        "si-user-selection-main-table": "si-user-selection-main-table",
      },
      dialogVisible: this.visible,
      initStyleFlag: false,
      dType:this.type,


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
    this.initData();


  },

  mounted() {
    // this.handleOnMounted()
  },

  beforeDestroy() {
    // this.unregisterFromRefList()
  },

  methods: {

    initData() {
      this.loadUser(() => this.initDefaultChecked());
      this.loadCurrentUser();
      this.loadConOrg();
      this.$nextTick(() => {
        if (this.dialogVisible) {
          this.initStyle();
        }
      });
    },
    initStyle() {
      let char = "px";
      //设置最多高度
      let maxHeight = 400;
      if (!!this.maxBodyHeight) {
        //如果是非字符串时
        if (!isNaN(this.maxBodyHeight)) {
          char = "";
          return Number(this.maxBodyHeight);
        }

        if (this.maxBodyHeight.indexOf("%") !== -1) {
          char = "%";
        }

        let charIndex = this.maxBodyHeight.indexOf(char);
        if (charIndex !== -1) {
          maxHeight = Number(this.maxBodyHeight.substring(0, charIndex));
        }
      }

      //分别设置几个高度
      let topCheckHeaderHeight = 50;
      let leftTreeHeaderHeight = 90;

      let leftTreeMaxHeight = maxHeight - (topCheckHeaderHeight + leftTreeHeaderHeight);
      let leftUserMaxHeight = leftTreeMaxHeight;
      let mainUserMaxHeight = maxHeight - (topCheckHeaderHeight) - 22;

      //设置对应高度
      document.querySelector(`.${this.contentClassName["si-user-selection-left-tree"]}`).style['height'] = leftTreeMaxHeight + char;
      document.querySelector(`.${this.contentClassName["si-user-selection-left-table"]}`).style['height'] = leftUserMaxHeight + char;
      document.querySelector(`.${this.contentClassName["si-user-selection-main-table"]}`).style['height'] = mainUserMaxHeight + char;

      this.initStyleFlag = true;

    },

    //初始化选中值
    initDefaultChecked(data) {
      if (data != null) {
        this.defaultChecked = data;
      }

      if (!isListEmpty(this.defaultChecked)) {
        if (this.type === "radio" && this.defaultChecked.length > 1) {
          this.$message.warning("单选只能传入一个值!");
          this.defaultChecked.splice(1, this.defaultChecked.length);
        }
        this.defaultChecked.forEach(userId => {
          if (!userId) {
            return
          }
          // eslint-disable-next-line no-prototype-builtins
          if (this.tableDataMap.hasOwnProperty(userId)) {
            this.multipleSelectionMap[userId] = this.tableDataMap[userId];
            //选中用户
            if (this.isMulti()) {
              this.userSelectionStateToFraUserState(userId, true);
              this.fraUserSelectionStateToUserState(userId, true);
            } else {
              this.setCurrentForUser(userId);
              this.setCurrentForFraUser(userId);
            }
          }
        });
        this.selectionMapToArray();
      }
    },

    setType(type){
      this.dType = type;
    },
    getTypeForColumn(){
      return this.dType === 'radio' ? 'index' : 'selection';
    },
    isMulti() {
      return this.dType !== "radio";
    },
    filterNode(value, data) {
      if (!value) return true;
      return data.label.indexOf(value) !== -1;
    },
    //map转array
    selectionMapToArray() {
      arrayClear(this.multipleSelection);
      for (let k in this.multipleSelectionMap) {
        let m = this.multipleSelectionMap[k];
        this.multipleSelection.push(m);
      }
    },
    //通过用户查找用户Id列表中对象
    getUserDataForUserId(userId) {
      // eslint-disable-next-line no-prototype-builtins
      if (this.tableDataMap.hasOwnProperty(userId)) {
        return this.tableDataMap[userId];
      }
      return null;
    },
    getCurrentUserDataForUserId(userId) {
      // eslint-disable-next-line no-prototype-builtins
      if (this.currentUserTableDataMap.hasOwnProperty(userId)) {
        return this.currentUserTableDataMap[userId];
      }
      return null;
    },
    getFraUserDataForUserId(userId) {
      // eslint-disable-next-line no-prototype-builtins
      if (this.fraOrgTableDataMap.hasOwnProperty(userId)) {
        return this.fraOrgTableDataMap[userId];
      }
      return null;
    },
    //通过架构中用户选中用户列表中用户
    fraUserSelectionStateToUserState(userId, checkState = true) {
      //fra转用户列表中数据
      let row = this.getUserDataForUserId(userId);
      if (row) {
        this.$refs[this.tableRefs.onlyUserTable].toggleRowSelection(row, checkState);
      }
    },
    //选中当前用户表中数据
    currentUserSelectionStateToUserState(userId, checkState = true) {
      //fra转用户列表中数据
      let row = this.getCurrentUserDataForUserId(userId);
      if (row) {
        this.$refs[this.tableRefs.currentUserTable].toggleRowSelection(row, checkState);
      }
    },
    //通过用户选中状态去调整组织架构列表中用户的选中状态
    userSelectionStateToFraUserState(userId, checkState = true) {
      //fra转用户列表中数据
      let row = this.getFraUserDataForUserId(userId);
      if (row) {
        this.$refs[this.tableRefs.fraUserTable].toggleRowSelection(row, checkState);
      }
    },

    //选择
    handleSelectionChangeForFra(checked) {
      if (this.type === "radio") {
        //将选择数据进行关联
        this.multipleSelection.forEach(u => {
          this.setCurrentForFraUser(u.userId);
        })
        return;
      }
      //去除选择的数据
      let fraCheckedMap = {};
      checked.forEach(c => {
        fraCheckedMap[c.userId] = c;
        this.fraUserSelectionStateToUserState(c.userId, true);
        this.currentUserSelectionStateToUserState(c.userId, true);
        // eslint-disable-next-line no-prototype-builtins
        if (!this.multipleSelectionMap.hasOwnProperty(c.userId)) {
          this.multipleSelectionMap[c.userId] = c;
        }
      })
      //去除选中的数据
      this.fraOrgTableData.forEach(user => {
        // eslint-disable-next-line no-prototype-builtins
        if (!fraCheckedMap.hasOwnProperty(user.userId)) {
          this.fraUserSelectionStateToUserState(user.userId, false);
          this.currentUserSelectionStateToUserState(user.userId, false);
          delete this.multipleSelectionMap[user.userId];
        }
      })
      this.selectionMapToArray();
    },
    handleSelectionChange(checked) {
      //多选时
      if (this.type === "radio") {
        return;
      }
      let fraCheckedMap = {};
      checked.forEach(c => {
        fraCheckedMap[c.userId] = c;
        this.userSelectionStateToFraUserState(c.userId, true)
        this.currentUserSelectionStateToUserState(c.userId, true)
        // eslint-disable-next-line no-prototype-builtins
        if (!this.multipleSelectionMap.hasOwnProperty(c.userId)) {
          this.multipleSelectionMap[c.userId] = c;
        }
      })
      //去除选中的数据
      this.tableData.forEach(user => {
        // eslint-disable-next-line no-prototype-builtins
        if (!fraCheckedMap.hasOwnProperty(user.userId)) {
          this.userSelectionStateToFraUserState(user.userId, false)
          this.currentUserSelectionStateToUserState(user.userId, false)
          delete this.multipleSelectionMap[user.userId];
        }
      })
      this.selectionMapToArray();
    },
    handleSelectionChangeForCurrentUser(checked) {
      //多选时
      if (this.type === "radio") {
        return;
      }
      let fraCheckedMap = {};
      checked.forEach(c => {
        fraCheckedMap[c.userId] = c;
        //选中
        this.userSelectionStateToFraUserState(c.userId, true)
        this.fraUserSelectionStateToUserState(c.userId, true);
        // eslint-disable-next-line no-prototype-builtins
        if (!this.multipleSelectionMap.hasOwnProperty(c.userId)) {
          this.multipleSelectionMap[c.userId] = c;
        }
      })
      //去除选中的数据
      this.currentUserTableData.forEach(user => {
        // eslint-disable-next-line no-prototype-builtins
        if (!fraCheckedMap.hasOwnProperty(user.userId)) {
          this.userSelectionStateToFraUserState(user.userId, false)
          this.fraUserSelectionStateToUserState(user.userId, false)
          delete this.multipleSelectionMap[user.userId];
        }
      })
      this.selectionMapToArray();
    },
    //单选
    setCurrentForUser(userId) {
      let row = this.getUserDataForUserId(userId);
      if (row) {
        this.$refs[this.tableRefs.onlyUserTable].setCurrentRow(row);
      }
    },
    //组织机构中的用户
    setCurrentForFraUser(userId) {
      let row = this.getFraUserDataForUserId(userId);
      if (row) {
        this.$refs[this.tableRefs.fraUserTable].setCurrentRow(row);
      }
    },
    //设置当前用户的状态为选中
    setCurrentForCurrentUser(userId) {
      let row = this.getCurrentUserDataForUserId(userId);
      if (row) {
        this.$refs[this.tableRefs.currentUserTable].setCurrentRow(row);
      }else{
        //取消
        this.$refs[this.tableRefs.currentUserTable].setCurrentRow();
      }
    },
    handleCurrentChange(val) {
      if (this.type === "multi") {
        return;
      }
      this.currentRow = val;
      let checked = [val];
      let fraCheckedMap = {};
      checked.forEach(c => {
        fraCheckedMap[c.userId] = c;
        this.setCurrentForFraUser(c.userId);
        this.setCurrentForCurrentUser(c.userId);
        // eslint-disable-next-line no-prototype-builtins
        if (!this.multipleSelectionMap.hasOwnProperty(c.userId)) {
          this.multipleSelectionMap[c.userId] = c;
        }
      })
      //去除选中的数据
      this.tableData.forEach(user => {
        // eslint-disable-next-line no-prototype-builtins
        if (!fraCheckedMap.hasOwnProperty(user.userId)) {
          delete this.multipleSelectionMap[user.userId];
        }
      })
      this.selectionMapToArray();
    },
    //单选- 当前用户
    handleCurrentChangeForCurrentUser(val) {
      if (this.type === "multi") {
        return;
      }
      if (!val) {
        return;
      }
      this.currentRow = val;
      let checked = [val];
      let fraCheckedMap = {};
      checked.forEach(c => {
        fraCheckedMap[c.userId] = c;
        this.setCurrentForFraUser(c.userId);
        this.setCurrentForUser(c.userId);
        // eslint-disable-next-line no-prototype-builtins
        if (!this.multipleSelectionMap.hasOwnProperty(c.userId)) {
          this.multipleSelectionMap[c.userId] = c;
        }
      })
      //去除选中的数据
      this.tableData.forEach(user => {
        // eslint-disable-next-line no-prototype-builtins
        if (!fraCheckedMap.hasOwnProperty(user.userId)) {
          delete this.multipleSelectionMap[user.userId];
        }
      })
      this.selectionMapToArray();
    },
    handleCurrentChangeForFra(val) {
      if (this.type === "multi") {
        return;

      }
      this.currentRow = val;
      let checked = [val];
      let fraCheckedMap = {};
      checked.forEach(c => {
        fraCheckedMap[c.userId] = c;
        this.setCurrentForUser(c.userId);
        this.setCurrentForCurrentUser(c.userId);
        // eslint-disable-next-line no-prototype-builtins
        if (!this.multipleSelectionMap.hasOwnProperty(c.userId)) {
          this.multipleSelectionMap[c.userId] = c;
        }
      })
      //去除选中的数据
      this.tableData.forEach(user => {
        // eslint-disable-next-line no-prototype-builtins
        if (!fraCheckedMap.hasOwnProperty(user.userId)) {
          delete this.multipleSelectionMap[user.userId];
        }
      })
      this.selectionMapToArray();
    },
    //删除标签
    handlerTagClick(tag) {
      let userId = tag.userId;
      //删除用户
      this.multipleSelection.forEach((u, index) => {
        if (u.userId === userId) {
          this.multipleSelection.splice(index);
          delete this.multipleSelection[userId];
          this.userSelectionStateToFraUserState(userId, false)
          this.fraUserSelectionStateToUserState(userId, false)
        }
      })
    },
    handleTabClick(tab) {
      // let tabName = tab.name;
    },
    handlerTreeNodeClick(node, node2) {
      if (node) {
        this.loadUserByConOrg(node.id);
      }
    },
    //随机获取标签颜色
    randomTagsType() {
      let index = Math.abs(Math.floor(10 * Math.random()) - 5)
      return this.tagsTypes[index];
    },
    //加载组织机构
    loadConOrg() {
      axios.get(`${this.getCtx()}base/general/fraOrg`).then(res => {
        if (res.status === 200 && res.data) {
          let resData = res.data;
          if (resData.code === 0) {
            let data = resData.data;
            this.fraOrgData = data;
          }
        }
      }).catch(res => {
      });
    },
    //加载用户
    loadUser(callback) {
      let self = this;
      axios.get(`${this.getCtx()}base/general/user?isNt=N`).then(res => {
        if (res.status === 200 && res.data) {
          let resData = res.data;
          if (resData.code === 0) {
            let data = resData.rows;
            arrayClear(self.tableData);
            arrayClear(self.filterTableData);
            data.forEach(d => {
              self.tableDataMap[d.userId] = d;
              self.tableData.push(d);
              self.filterTableData.push(d);
              // eslint-disable-next-line no-prototype-builtins
              if (self.multipleSelectionMap.hasOwnProperty(d.userId)) {
                self.fraUserSelectionStateToUserState.call(self, d.userId, true);
              }
            })
          }
        }
        callback && typeof (callback) === "function" && callback.call(self);
      }).catch(res => {
      });

    },
    loadCurrentUser() {
      let self = this;
      axios.get(`${this.getCtx()}base/general/current_user`).then(res => {
        if (res.status === 200 && res.data) {
          let resData = res.data;
          if (resData.code === 0) {
            let data = [resData.data];
            arrayClear(self.currentUserTableData);
            data.forEach(d => {
              self.currentUserTableDataMap[d.userId] = d;
              self.currentUserTableData.push(d);
              // eslint-disable-next-line no-prototype-builtins
              if (self.multipleSelectionMap.hasOwnProperty(d.userId)) {
                self.currentUserSelectionStateToUserState.call(self, d.userId, true);
              }
            })
          }
        }
      }).catch(res => {
      });

    },
    //根据组织架构加载用户
    loadUserByConOrg(fraOrgId) {
      let self = this;
      axios.get(`${this.getCtx()}base/general/user_by_fra_org?fraOrgId=${fraOrgId}`).then(res => {
        if (res.status === 200 && res.data) {
          let resData = res.data;
          if (resData.code === 0) {
            let data = resData.data;
            arrayClear(self.fraOrgTableData);
            data.forEach(u => {
              let user = {};
              user.userId = u.userId;
              user.name = u.userName;
              if (u.dept) {
                user.dept = u.dept.deptName;
              }
              //如果在选择列表中时进行选中
              self.fraOrgTableDataMap[u.userId] = u;
              self.fraOrgTableData.push(u);
              // eslint-disable-next-line no-prototype-builtins
              if (self.multipleSelectionMap.hasOwnProperty(u.userId)) {
                self.userSelectionStateToFraUserState.call(self, u.userId, true);
              }
            })

          }
        }
      }).catch(res => {
      });
    },
    handlerFraTreeExpand() {
      this.fraTreeExpand = !this.fraTreeExpand
      this.buildData();
    },
    buildData() {
      for (let i = 0; i < this.$refs.fraOrgTreeNode.store.root.childNodes.length; i++) {
        this.$refs.fraOrgTreeNode.store.root.childNodes[i].expanded = this.fraTreeExpand;
      }
    },
    rowClassName({row}) {
      // eslint-disable-next-line no-prototype-builtins
      if (row.hasOwnProperty('showClass')) {
        return row.showClass;
      }
      return "";
    },
    listData() {
      let self = this;
      let tableBodyEle = self.$refs[this.tableRefs.onlyUserTable].$refs['bodyWrapper'];
      let rows = tableBodyEle.querySelectorAll(".el-table__row");
      let val = this.filterText;
      this.tableData.forEach((data, index) => {
        let bool = !val || data.userName.toLowerCase().includes(val.toLowerCase());
        if (!bool) {
          data.showClass = "si-uer-hidden-row";
          rows[index].classList.add("si-uer-hidden-row");
        } else {
          rows[index].classList.remove("si-uer-hidden-row")
        }
      })
    },
    handleClose(done) {
      if (done === "yes") {
        //关闭
        this.dialogVisible = false;
        this.$emit("user-selection-data", this.multipleSelection, this.multipleSelectionMap);
        return;
      }
      done();
      // this.$confirm('确认关闭？')
      //   .then(_ => {
      //     done();
      //   })
      //   .catch(_ => {
      //   });
    }
  },
  watch: {
    filterFraText(val) {
      this.$refs.fraOrgTreeNode.filter(val);
    },
    filterText(val) {
      this.listData();
    },
    dialogVisible(val) {

      if (val && !this.initStyleFlag) {
        setTimeout(() => {
          this.initStyle();
        }, 600)
      }
    }
  },
}
</script>

<style lang="scss" scoped>
@import "../../../styles/global.scss"; //* form-item-wrapper已引入，还需要重复引入吗？ *//

.full-width-input {
  width: 100% !important;
}

.si-user-selection-root ::v-deep .el-dialog__body {
  padding-top: 0;
}

::v-deep .si-user-selection {
  height: 400px;
  overflow: auto;
}

/*左侧滚动树*/
::v-deep .si-user-selection-left-tree {
  height: 400px;
  overflow: auto;
}

/*左侧滚动树*/
::v-deep .si-user-selection-left-table {
  height: 400px;
  overflow: auto;
}

::v-deep .si-user-selection-main-table {
  max-height: 400px;
  overflow: auto;
}

::v-deep .el-table .si-uer-hidden-row {
  display: none;
}

.si-select-user {
  border: 1px solid #e0e0e0;
  min-height: 30px;
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
