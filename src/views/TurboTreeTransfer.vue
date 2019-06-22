<template>
  <div class="root">
    <div class="left">
      <el-checkbox @change='fromAllBoxChangePerson2'></el-checkbox>
      <el-tree
        lazy
        :load="loadNode1"
        ref='from-tree-person' :props="props" accordion :filter-node-method="filterPersonName" node-expand
        :data="personFromData" show-checkbox node-key="id">
      </el-tree>
    </div>
    <div class="center">
      <el-button type="primary" @click="addToAimsPerson2" icon="el-icon-arrow-right">添加
      </el-button>
      <!--                  <el-button type="primary" @click='removePerson2' :disabled="to_disabled_person" icon="el-icon-arrow-left">{{$t("txt_item_291")}}-->
      <el-button type="primary" @click='removePerson2' icon="el-icon-arrow-left">删除
      </el-button>
    </div>
    <div class="right">
      <el-checkbox
        @change="toAllBoxChangePerson2"></el-checkbox>
      <el-tree
        :load="loadNode1" lazy :props="props"
        :filter-node-method="fanxiangfilterPersonName"
        slot='to' ref='to-tree-person' :data="personFromData" show-checkbox node-key="id">
      </el-tree>
    </div>
  </div>
</template>
<script>
// import { cloneDeepWith } from 'lodash'
import _ from 'lodash'
// import { reqCycleList } from '@/api'
export default {
  data () {
    return {
      props: {
        label: 'name',
        isLeaf: this.isLeaf
      }
    }
  },
  props: {
    personFromData: {
      type: Array,
      default: () => []
    }
  },
  created () {
  },
  mounted () {
    this.initTree()
  },
  /* eslint-disable */
    methods: {
      loadNode1 (node, resolve) {
        setTimeout(() => {
          if (node.level === 0) {
            resolve(node.data)
            this.initTree()
          } else {
            resolve(node.data.children)
            this.initTree()
          }
        }, 10)
      },
      isLeaf (data, node) {
        return data.id !== data.group_id
      },
      // 数据变动后，过滤数据。根据过滤函数‘filterPersonName’和‘fanxiangfilterPersonName’返回值，确定左右两个树的节点 显示还是隐藏
      initTree () {
        this.$refs['from-tree-person'].filter('')
        this.$refs['to-tree-person'].filter('')
      },

      getCheckedData (checkedArr, checkedGroupIDs, arr) {
        if (arr) {
          arr.forEach(item => {
            // 这里是 组
            if (item.id === item.group_id) {
              // 半选
              if (item.isHalfDisabled) {

              }
              // 全选
              if (item.isDisabled) {
                checkedGroupIDs.push(item.id)
              }
              this.getCheckedData(checkedArr, checkedGroupIDs, item.children)
            } else if (item.isDisabled || _.findIndex(checkedGroupIDs, function (o) {
              return o === item.group_id
            }) !== -1) {
              // 这里面是 人
              checkedArr.push(item.id)
            }
          })
        }
      },
      findAllCheckedPersonsID () {
        let checkedPersonIDs = []
        let checkedGroupIDs = []
        this.getCheckedData(checkedPersonIDs, checkedGroupIDs, this.personFromData)
        checkedPersonIDs = _.join(checkedPersonIDs, ',')
        return checkedPersonIDs
      },
      //   重新初始化最开始的数据
      resetPersonData (arr) {
        if (arr) {
          arr.forEach(item => {
            this.visiableNode(item)
            if (this.isGroup(item)) {
              this.halfVisiableNode(item)
              this.resetPersonData(item.children)
            }
          })
        }
      },
      // 人员树重置
      initData2 () {
        this.resetPersonData(this.personFromData)
        this.initTree()
      },

      // input筛选人员名，反向过滤，被禁用的条件都过滤
      fanxiangfilterPersonName (value, data) {
        if (data.isDisabled || data.isHalfDisabled) {
          return true
        }
      },
      // input筛选人员名
      filterPersonName (value, data) {
        if (data.isDisabled) {
          return false
        } else if (!value) return true
        return data.name.indexOf(value) !== -1
      },

      // 选中全部子
      checkedAllChildren (arr) {
        if (arr) {
          arr.forEach(item => {
            this.invisiableNode(item)
            if (this.isGroup(item)) {
              this.checkedAllChildren(item.children)
            }
          })
        }
      },
      // 全部子取消选中
      NotcheckedAllChildren (arr) {
        if (arr) {
          arr.forEach(item => {
            this.visiableNode(item)
            if (this.isGroup(item)) {
              this.NotcheckedAllChildren(item.children)
            }
          })
        }
      },
      // 是否是group
      isGroup (item) {
        return item.id === item.group_id
      },
      addToAimsPerson2 () {
        // element-ui:4.4.1写法
        // let checkedNodes = this.$refs['from-tree-person'].getCheckedNodes(false, true)
        //element-ui:4.9.2写法
        let checkedNodes = this.$refs['from-tree-person'].getCheckedNodes(false, false)
        checkedNodes.forEach(item => {
          this.invisiableNode(item)
          if (this.isGroup(item)) {
            this.checkedAllChildren(item.children)
          }
        })
        let halfCheckedNodes = this.$refs['from-tree-person'].getHalfCheckedNodes()
        halfCheckedNodes.forEach(item => {
          this.halfInvisiableNode(item)
        })
        this.initTree()
        this.changeCheckedKeys()
      },
      // 左侧半选节点隐藏，实际不会隐藏，但是右侧的tree会显示半选节点
      halfInvisiableNode (item) {
        this.$set(item, 'isHalfDisabled', true)
      },
      // 禁用节点，
      // 左侧隐藏选取的节点
      invisiableNode (item) {
        this.$set(item, 'isDisabled', true)
      },

      // 左侧显示某节点，
      visiableNode (item) {
        this.$set(item, 'isDisabled', false)
      },
      // 左侧显示某半选节点，
      halfVisiableNode (item) {
        this.$set(item, 'isHalfDisabled', false)
      },
      removePerson2 () {
        // element-ui:4.4.1写法
        // let checkedNodes = this.$refs['to-tree-person'].getCheckedNodes(false, true)
        //element-ui:4.9.2写法
        let checkedNodes = this.$refs['to-tree-person'].getCheckedNodes(false, false)
        checkedNodes.forEach(item => {
          this.visiableNode(item)
          // 如果是组，全选和半选 标记 都关闭
          if (item.id === item.group_id) {
            this.halfVisiableNode(item)
            this.NotcheckedAllChildren(item.children)
          }
        })
        //半选节点关闭//element-ui:4.9.2写法
        let halfCheckedNodes = this.$refs['to-tree-person'].getHalfCheckedNodes()
        halfCheckedNodes.forEach(item => {
          this.halfVisiableNode(item)
        })
        this.initTree()
        this.changeCheckedKeys()
      },

      //对树完成全选/ 全不选
      checkedOrNotAllData (tree, val) {
        this.personFromData.forEach(item => {
          tree.setChecked(item, val, true)
        })
      },
      // 左侧数据 全选或全不选
      fromAllBoxChangePerson2 (val) {
        this.checkedOrNotAllData(this.$refs['from-tree-person'], val)
      },
      // 右侧侧数据 全选或全不选
      toAllBoxChangePerson2 (val) {
        this.checkedOrNotAllData(this.$refs['to-tree-person'], val)
      },
      //向父返回选中keys
      changeCheckedKeys(){
        this.$emit('changeCheckedKeys',this.findAllCheckedPersonsID())
      }
    },
    watch: {
      personFromData(){

      }
    }
  }
</script>

<style lang='scss' scoped>
  .root {
    display: flex;
    .left,.right{
      width: 300px;
    }
  }
</style>
