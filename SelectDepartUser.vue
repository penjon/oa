<template>
  <el-dialog
    :append-to-body="true"
    :before-close="cancel"
    :close-on-click-modal="false"
    :title="title"
    :visible.sync="boxDialogVisible"
    class="tree-dialog-form scrollbar selectBox"
    heihgt="607px"
    width="686px"
  >
    <div class="box">
      <div class="left-box">
        <div class="left-top">
          <el-input
            :placeholder="searchConfig.title ? searchConfig.title : '请输入'"
            @input="leftInputChange"
            @keyup.enter.native="enter(searchConfig.params[searchConfig.key])"
            class="vc-search-input"
            size="small"
            v-if="searchConfig.show === false ? false : true"
            v-model="searchConfig.params[searchConfig.key]"
          >
            <el-button slot="append" icon="el-icon-search" @click="enter(searchConfig.params[searchConfig.key])"></el-button>
          </el-input>
        </div>
        <div class="left-bottom">
          <div style="height: 93%">
            <el-form class="personnel-form-top" label-width="90px" size="small">
              <div
                class="vc-page vc-page-parts"
                style="
                  position: relative;
                  overflow: hidden;
                  padding-bottom: 20px;
                  margin-right: 0px;
                "
              >
                <div
                  class="tree-dialog scrollbar"
                  style="width: 600px; background: white"
                >
                  <el-tree
                    :data="treeConfig.insideTreeData"
                    :default-expanded-keys="treeConfig.expandedKeys"
                    :empty-text="treeConfig.treeEmptyText"
                    :expand-on-click-node="true"
                    :key="treeKey"
                    :props="treeConfig.defaultProps"
                    :ref="'roleTree'"
                    :render-content="renderContent"
                    :filter-node-method="filterNode"
                    @check-change="treeConfig.handleSelectionChange"
                    check-strictly
                    class="org-tree"
                    highlight-current
                    node-key="treeId"
                    render-after-expand
                    show-checkbox
                    v-if="!showList && listStyle === 'tree' && indep"
                  ></el-tree>
                  <el-tree
                    :data="treeConfig.insideTreeData"
                    :default-expanded-keys="treeConfig.expandedKeys"
                    :empty-text="treeConfig.treeEmptyText"
                    :expand-on-click-node="true"
                    :key="treeKey"
                    :props="treeConfig.defaultProps"
                    :ref="'roleTree'"
                    :render-content="renderContent"
                    :filter-node-method="filterNode"
                    @check-change="treeConfig.handleSelectionChange"
                    class="org-tree"
                    highlight-current
                    node-key="treeId"
                    render-after-expand
                    show-checkbox
                    v-if="!showList && listStyle === 'tree' && !indep"
                  ></el-tree>
                  <el-checkbox-group
                    class="checkbox"
                    v-if="showList && leftList.items.length !== 0"
                    v-model="leftList.default"
                  >
                    <div
                      :key="idx"
                      class="checkBox-box"
                      v-for="(item, idx) in leftList.items"
                    >
                      <div class="vm-line icon-left">
                        <el-checkbox
                          :label="item.id"
                          @change="checked => leftListChange(checked, item)"
                        >
                          <i
                            class="checkedIcon"
                            :class="iconConfig.childrenType.icon"
                            style="color: rgba(170, 207, 255, 1)"
                          ></i>
                          <span>{{ item[searchConfig.key] }}</span>
                        </el-checkbox>
                      </div>
                    </div>
                  </el-checkbox-group>
                  <div
                    style="text-align: left; padding-left: 20px;"
                    v-else-if="showList && leftList.items.length === 0"
                  >
                    暂无数据
                  </div>
                </div>
              </div>
            </el-form>
          </div>
        </div>
      </div>
      <div class="right-box" style="float: right; height: 100%">
        <div class="right-top">
          <div class="top-box">
            <span
              >已选择&nbsp;{{
                rightList.items.length ? `(${rightList.items.length})` : ''
              }}</span
            >
            <span
              v-show="rightList.items.length > 0"
              @click="removeCheck"
              class="color-primary"
              style="cursor: pointer"
              >清空</span
            >
          </div>
        </div>
        <div class="right-bottom scrollbar">
          <p
            v-show="rightList.items.length === 0"
            style="font-size: 14px; color: #ccc; text-align: center;     padding-top: 110px;"
          >
            请从左侧列表选择需要添加的人员
          </p>
          <ul class="checkbox">
            <li
              :key="idx"
              class="checkBox-box"
              v-for="(item, idx) in rightList.items"
              v-show="item.type !== iconConfig.parentType.key"
            >
              <div class="icon-left-box">
                <i class="vc-icon" :class="iconConfig.childrenType.icon"></i>
                <span class="valueBox">{{
                  item[rightKey] || item[rightKeyQ]
                }}</span>
              </div>
              <div class="vm-line icon-right">
                <i
                  class="el-icon-circle-close"
                  style="color: #dfe7f1"
                  v-on:click="clickIcon(item)"
                ></i>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <div class="dialog-footer" style="text-align: right; padding: 20px">
      <el-button @click="cancel" size="small">取消</el-button>
      <el-button
        :disabled="rightList.items.length === 0"
        @click="save"
        size="small"
        type="primary"
        >保存</el-button
      >
    </div>
  </el-dialog>
</template>

<script>
import { clone } from '^^/utils'
export default {
  name: 'SelectBox',
  components: {},
  props: {
    title: {
      type: String,
      default() {
        return ''
      }
    },
    searchConfig: {
      type: Object,
      default() {
        return {
          title: '请输入',
          http: null,
          show: true,
          key: 'username',
          params: {}
        }
      }
    },
    rightKey: {
      type: String,
      default() {
        return 'name'
      }
    },
    rightKeyQ: {
      type: String,
      default() {
        return ''
      }
    },
    boxDialogVisible: {
      type: Boolean,
      default() {
        return false
      }
    },
    checkedNode: {
      type: Array,
      default() {
        return []
      }
    },
    allNode: {
      type: Array,
      default() {
        return []
      }
    },
    listStyle: {
      type: String,
      default() {
        return 'tree'
      }
    },
    defaultProps: {
      type: Object,
      default() {
        return {
          id: 'treeId',
          label: 'name',
          children: 'children',
          isLeaf: 'isLeaf'
        }
      }
    },
    iconConfig: {
      type: Object,
      default() {
        return {
          parentType: {
            key: 'pre',
            icon: 'el-icon-folder'
          },
          childrenType: {
            key: 'permission',
            icon: 'el-icon-user'
          }
        }
      }
    },
    indep: {
      type: Boolean,
      default() {
        return false
      }
    }
  },
  data() {
    return {
      defaultPer: {
        disabled: data => {
          if (
            data.type === this.iconConfig.parentType.key &&
            data.children &&
            data.children.length === 0
          ) {
            return true
          }
        }
      },
      disabled: false,
      showList: false,
      treeKey: '1',
      allNodes: clone(this.allNode),
      insideAllNode: [],
      insideChecked: [],
      checkList: [],
      expandedKeys: [],
      treeEmptyText: ' ',
      // 索引值
      index: 0,
      // insideTreeData
      insideTreeData: [],
      inSideDefaultProps: {
        id: 'id',
        label: 'text',
        children: 'children',
        isLeaf: 'isLeaf'
      },
      treeConfig: {
        insideTreeData: [],
        expandedKeys: [],
        treeEmptyText: ' ',
        handleSelectionChange: this.handleSelectionChange
      },
      // 最后提交
      addList: [],
      deleteList: [],
      // 右侧多选列表
      rightList: {
        items: [],
        default: []
      },
      // 右侧多选列表
      leftList: {
        items: [],
        default: []
      }
    }
  },
  computed: {},
  mounted() {},
  created() {},

  watch: {
    boxDialogVisible(value) {
      if (value) {
        this.init()
        this.disabled = false
      }
    }
  },

  methods: {
    /*
    leftInputChange
    */
    leftInputChange(value) {
      if (this.listStyle === 'tree' && value.length === 0) {
        this.showList = false
        this.$nextTick(() => {
          this.$refs.roleTree.setCheckedKeys(this.leftList.default)
        })
      }
    },
    /*

    */
    onFocus() {
      const search = document.getElementsByClassName('select-box-search')[0]
      search.onclick = () => this.search()
    },
    search() {
      let flage = true
      Object.keys(this.searchConfig.params).forEach(i => {
        this.searchConfig.params[i] = this.searchConfig.params[i].trim()
        if (this.searchConfig.params[i] === '') {
          flage = false
        }
      })
      if (flage) {
        this.searchConfig
          .http({
            data: this.searchConfig.params
          })
          .then(res => {
            this.leftList.items = res.data
            this.judgeOldNode(this.leftList.items, this.insideChecked)
            this.showList = true
          })
      }
    },
    /*
      回车
    */
    enter() {
      return this.search()
    },
    /*
      初始化结构
    */
    init() {
      this.initData()
      this.disable(this.insideAllNode, this.insideChecked)
      this.initLeftData()
      this.initRightData()
      this.$nextTick(() => {
        this.$refs.roleTree.filter()
      })
    },
    // 初始化数据
    initData() {
      if (this.$refs.roleTree && this.$refs.roleTree[0]) {
        this.$refs.roleTree.setCheckedKeys([])
      }
      this.rightList.items = []
      this.rightList.default = []
      this.insideChecked = []
      this.insideAllNode = []
      this.insideAllNode = clone(this.allNodes)
      this.insideChecked = clone(this.checkedNode)
      this.leftList.default = this.insideChecked.map(i => i.id)
    },
    // 初始化左侧选人组件
    initLeftData() {
      this.insideListStyle = this.judgeStyle()
      if (this.insideListStyle === 'tree') {
        this.initTree(this.insideAllNode, this.insideChecked)
      } else {
        this.initList(this.insideAllNode, this.insideChecked)
      }
    },
    judgeStyle() {
      return this.listStyle
    },
    initTree(all) {
      this.getInsideTreeData(all)
    },
    initList(all) {
      this.leftList.items = all
      this.showList = true
    },
    filterNode(value, data, node) {
      if (data.type === 'pre' && !node.children) { return false }
      return true
    },
    /**
     * 节点禁用  取交集
     */
    disable(all, checked) {
      this.judgeOldNode(all, checked)
      this.rightList.items = all.filter(i => i.old)
    },
    judgeOldNode(all, checked) {
      all.forEach(x => {
        if (checked.some(y => y === x.id)) {
          x.disabled = true
          x.old = true
        }
      })
    },
    /**
     * 构建右侧列表
     */
    initRightData() {
      this.rightList.items = this.insideAllNode.filter(i => i.old && i.type !== 'pre')
    },
    /**
     * 左侧多选框改变
     */
    leftChange(data) {
      const arr = this.insideAllNode.filter(i => {
        if (data.some(x => x === i.id)) {
          return i
        }
      })
      this.rightList.items = this.rightList.items.concat(arr)
      const obj = {}
      this.rightList.items = this.rightList.items.reduce((cur, next) => {
        if (!obj[next.id]) {
          cur.push(next)
        }
        return cur
      }, [])
    },
    /**
     * 左侧列表改变
     */
    leftListChange(data, node) {
      if (!data) {
        this.rightList.items.splice(
          this.rightList.items.findIndex(i => i.id === node.id),
          1
        )
      } else {
        node[this.rightKey] = node[this.searchConfig.key]
        this.rightList.items.push(node)
      }
    },
    /**
     * 批量移除
     */
    removeCheck() {
      if (!this.showList) {
        this.removeTree()
      } else {
        this.removeList()
      }
      this.rightList.items = []
      this.rightList.default = []
      this.leftList.default = []
    },
    /**
     * 添加到 deleteList
     */

    removeList() {
      this.leftList.default = []
    },
    removeTree() {
      this.$refs.roleTree.setCheckedKeys([])
    },
    /**
     * 节点禁用函数
     */
    disabledFn(data) {
      if (this.rightList.items.filter(i => i.id === data.id).length !== 0) {
        return true
      }
      return false
    },
    /*
      单选删除
    */
    clickIcon(item) {
      this.rightList.items.splice(
        this.rightList.items.findIndex(i => i.id === item.id),
        1
      )
      this.leftList.default.splice(
        this.leftList.default.findIndex(i => i === item.id),
        1
      )
      this.rightList.default.splice(
        this.rightList.default.findIndex(i => i === item.id),
        1
      )
      if (!this.showList) {
        this.deleteTree(item)
      } else {
        this.deleteleftList(item)
      }
    },
    deleteleftList(item) {
      const arr = this.leftList.default
      arr.splice(
        arr.findIndex(i => i === item.id),
        1
      )
    },
    deleteTree(item) {
      this.$nextTick(() => {
        this.$refs.roleTree.setChecked(item.treeId, false)
      })
    },
    /**
     * 左侧树状图选择
     */
    handleSelectionChange(data, status) {
      const allcheckedNode = this.$refs.roleTree.getCheckedNodes()
      const checkedNode = allcheckedNode.filter(
        i => i.type !== this.iconConfig.parentType.key
      )
      this.leftList.default = checkedNode.map(i => i.id)
      if (status) {
        const array = this.rightList.items.concat(checkedNode)
        const obj = {}
        this.rightList.items = array.reduce((cur, next) => {
          obj[next.id] ? '' : (obj[next.id] = true && cur.push(next))
          return cur
        }, [])
      } else {
        this.rightList.items = this.rightList.items.filter(i =>
          checkedNode.some(x => x.id === i.id)
        )
      }
    },
    clear() {
      this.$refs.roleTree.setCheckedKeys([])
      this.disabled = true
    },

    /**
     * 获取树结构
     */
    getInsideTreeData(lists) {
      this.showList = false
      this.treeConfig.expandedKeys = ['root']
      this.treeConfig.defaultProps = Object.assign(
        this.defaultProps,
        this.defaultPer
      )
      this.addItemIcon(lists)
      this.treeConfig.insideTreeData = this.transToTreeData(lists)
      const list_t = []
      if (this.insideChecked.length !== 0) {
        for (let i = 0; i < this.insideChecked.length; i++) {
          list_t.push(this.insideChecked[i])
        }
        this.treeConfig.expandedKeys = list_t.map(item => `u-${item}`)
      } else {
        const expandedKeys = []
        this.getFirst(lists, expandedKeys)
        // this.treeConfig.expandedKeys = [treeConfig.insideTreeData[0].id];
      }
      this.$nextTick(() => {
        this.$refs.roleTree.setCheckedKeys(list_t.map(item => `u-${item}`))
      })
    },

    /**
     * 转化为el-tree树形结构数据
     */
    transToTreeData(arr) {
      let list = JSON.parse(JSON.stringify(arr))
      if (!list || !list.length) return []
      var map = {}
      for (var item of list) {
        map[item.treeId] = item
      }
      var nodes = []
      for (var lis of list) {
        var p = map[lis.pid]
        if (!p) {
          nodes.push(lis)
          continue
        }
        p.children || (p.children = [])
        p.children.push(lis)
      }
      return nodes
    },

    // 添加图标
    addItemIcon(lists) {
      for (let i = lists.length - 1; i >= 0; i--) {
        // 添加图标
        if (lists[i].type === this.iconConfig.parentType.key) {
          lists[i].icon = this.iconConfig.parentType.icon
        } else {
          lists[i].icon = this.iconConfig.childrenType.icon
        }
      }
    },
    /*
      递归找第一个 type = 'permission'
    */
    getFirst(data, expandedKeys) {
      for (const i in data) {
        if (data[i]) {
          const node = data[i]
          if (node.type !== this.iconConfig.parentType.key) {
            expandedKeys.push(node)
            return
          } else if (node.children && node.children.length !== 0) {
            this.getFirst(node.children, expandedKeys)
          }
        }
      }
    },
    /**
     * 保存
     */
    save() {
      this.disabled = true
      this.addList = this.getAddList()
      this.deleteList = this.getdeleteList()
      this.searchConfig.params = {}
      const allItems = this.rightList.items.map(item => item.id)
      this.$emit('save', this.addList, this.deleteList, allItems)
    },
    closeDisabled() {
      this.disabled = false
    },
    getAddList() {
      const add = this.rightList.items.filter(i => !i.old).map(x => x.id)
      return add
    },
    getdeleteList() {
      const oldList = [...this.insideChecked]
      const newList = this.rightList.items.filter(i => i.old).map(x => x.id)
      const deleteList = oldList
        .filter(i => !newList.some(x => x === i))
        .map(i => i)
      return deleteList
    },
    /**
     * 取消
     */
    cancel() {
      this.searchConfig.params = {}
      this.$emit('close', false)
    },
    /**
      左侧树图标
    */
    renderContent(h, { node, data }) {
      return (
        <span class="treeSpan">
          <i class={data.icon}></i>
          <span> {node.label}</span>
        </span>
      )
    }
  }
}
</script>

<style lang="scss">
li {
  margin: 0;
  padding: 0;
  list-style: none;
}
.selectBox {
  /* 滚动条滑块 */
  .tree-dialog::-webkit-scrollbar-thumb {
    /*滚动条里面小方块*/
    border-radius: 5px;
    box-shadow: inset 0 0 5px #dfe7f1;
    background: #dfe7f1;
  }
  .right-bottom::-webkit-scrollbar-thumb {
    /*滚动条里面小方块*/
    border-radius: 5px;
    box-shadow: inset 0 0 5px #dfe7f1;
    background: #dfe7f1;
  }
  li {
    margin: 0;
    padding: 0;
    list-style: none;
  }
  .vm-line {
    display: inline-block;
  }
  .box {
    overflow: hidden;
    height: 496px;
    border-bottom: 1px solid #dfe7f1;
    //左侧
    .left-box {
      overflow: hidden;
      height: 100%;
      width: 43%;
      float: left;
      border-right: 1px solid #dfe7f1;
      padding-bottom: 70px;
      .left-top {
        height: 67px;
        padding: 0 23px 0 20px;
        display: flex;
        justify-content: center;
        align-items: center;
      }
      .left-bottom {
        padding: 0 0 0 17px;
        height: 100%;
        overflow: auto;
        .el-tabs__item {
          line-height: 19px;
          height: 27px;
        }
      }
    }
    // 右侧
    .right-box {
      width: calc(57% - 1px);
      overflow: auto;
      .right-top {
        text-align: left;
        display: flex;
        justify-content: space-between;
        align-items: center;
        height: 58px;
        width: 100%;
        .top-box {
          width: 100%;
          margin: 0px 20px;
          height: 100%;
          text-align: left;
          display: flex;
          justify-content: space-between;
          align-items: center;
        }
      }
      .right-bottom {
        height: calc(100% - 58px);
        .icon-right {
          text-align: right;
          width: 80px;
          i {
            height: 18px;
            width: 18px;
            font-size: 18px;
            padding-right: 18px;
          }
          .el-icon-circle-close {
            cursor: pointer;
            &:hover {
              color: rgb(170, 207, 255);
            }
          }
        }
      }
    }
    .checkbox {
      overflow: hidden;
      margin-top: -10px;
      padding: 0 20px 0 0;
      .checkBox-box {
        text-align: left;
        display: flex !important;
        color: #606266;
        font-weight: 500;
        font-size: 14px;
        display: inline-block;
        text-overflow: ellipsis;
        overflow: hidden;
        white-space: nowrap;
        -moz-user-select: none;
        -webkit-user-select: none;
        -ms-user-select: none;
        user-select: none;
        justify-content: space-between;
        margin: 10px 0 0px 30px;
        box-sizing: border-box;
        .vc-icon {
          margin-right: 5px;
        }
        .icon-right {
          float: right;
        }
        .icon-left-box {
          display: flex;
        }
        .valueBox {
          width: 180px;
          display: inline-block;
          text-overflow: ellipsis;
          overflow: hidden;
          white-space: nowrap;
          line-height: 14px;
        }
      }
    }
  }
  .treeSpan {
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
  }
  .vm-line {
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
  }
  //定义tab
  .button-small {
    padding: 7px 20px;
    font-size: 14px;
  }
  .el-tabs--top {
    height: 100%;
    margin-top: -20px;
  }

  .el-tabs__content {
    height: 100%;
  }
  .icon-left {
    width: 210px;
  }
}
// 弹出框样式2—为了匹配展示tree的样式。_cdw
.tree-dialog-form {
  .el-tabs__nav-scroll {
    padding: 0 23px;
  }
  display: flex;
  justify-content: center;
  align-items: center;
  .el-dialog {
    height: 607px;
    .el-dialog__footer {
      margin-left: 15px !important;
    }
    .el-dialog__header {
      border-bottom: 1px solid #dfe7f1;
      padding: 10px 17px;
      .el-dialog__headerbtn {
        top: auto;
      }
    }
    margin: 0 !important;
    border-radius: 5px;
    .el-dialog__title {
      width: 64px;
      height: 21px;
      font-size: 16px;
      font-family: Microsoft YaHei;
      font-weight: 400;
      line-height: 21px;
      color: rgba(68, 68, 68, 1);
      opacity: 1;
    }
    .el-dialog__body {
      height: calc(100% - 70px);

      padding: 0;
      .el-form {
        height: 100%;
        padding-bottom: 2px;
        .el-form-item .show-password .el-input__inner {
          padding-right: 50px;
        }
      }
    }
    .el-dialog__footer {
      text-align: right;
    }
  }

  &.append-to-body {
    font-family: 'Microsoft YaHei';
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    .el-dialog__header {
      text-align: left !important;
    }
    .el-main {
      min-height: 200px;
      padding: 0;
    }
  }
}
.el-dialog__header {
  text-align: left !important;
}
// 弹出框中展示树_cdw
.tree-dialog {
  width: 237px;
  height: 100%;
  margin-right: 0px;
  .org-tree {
    width: 100%;
    display: inline-block !important;
    .el-tree-node__content {
      height: 34px;
      line-height: 34px;
      padding-right: 5px;
    }
  }
}
</style>

