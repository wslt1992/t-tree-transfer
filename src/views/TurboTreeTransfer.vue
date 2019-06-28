<template>
  <div class="root" :style="`height:${height+100}`">
    <div class="left" :style="`height:${height}`">
      <h3 v-if="titles.length!==0">{{titles[0]}}</h3>
      <div class="left-main">
        <el-checkbox @change='fromAllBoxChangePerson2'>全选</el-checkbox>
        <el-tree
          lazy
          :load="loadNode1"
          ref='from-tree-person' :props="props" accordion :filter-node-method="filterPersonName" node-expand
          :data="fromData" show-checkbox node-key="id">
        </el-tree>
      </div>
    </div>
    <div class="center">
      <div>
        <el-button type="primary" @click="addToAimsPerson2" icon="el-icon-arrow-right">添加</el-button>
      </div>
      <div>
        <el-button type="primary" @click='removePerson2' icon="el-icon-arrow-left">删除</el-button>
      </div>
    </div>
    <div class="right" :style="`height:${height}`">
      <h3 v-if="titles.length!==0">{{titles[1]}}</h3>
      <div class="right-main">
        <el-checkbox @change="toAllBoxChangePerson2">全选</el-checkbox>
        <el-tree
          :load="loadNode1" lazy :props="props"
          :filter-node-method="fanxiangfilterPersonName"
          slot='to' ref='to-tree-person' :data="fromData" show-checkbox node-key="id">
        </el-tree>
      </div>
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
    fromData: {
      type: Array,
      default: () => []
    },
    height: {
      type: String,
      default: '500px'
    },
    titles: {
      type: Array,
      default: () => ['源数据', '结果']
    }
  },
  watch: {},
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
        }, 1)
      },
      isLeaf (data, node) {
        return data.id !== data.group_id
      },
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
        this.getCheckedData(checkedPersonIDs, checkedGroupIDs, this.fromData)
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
        this.resetPersonData(this.fromData)
        this.initTree()
        this.emitToParentChangeCheckedDataNull()
      },
      //清除选中的数据
      emitToParentChangeCheckedDataNull(){
        this.$emit('changeCheckedKeys',[])
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

      /******************other start*******************/
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
      isGroup (item) {
        return item.id === item.group_id
      },
      addToAimsPerson2 () {
        let checkedNodes = this.$refs['from-tree-person'].getCheckedNodes()
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
        this.emitToParentChangeCheckedData()
      },
      emitToParentChangeCheckedData(){
        this.$emit('changeCheckedKeys',this.findAllCheckedPersonsID())
      },
      halfInvisiableNode (item) {
        this.$set(item, 'isHalfDisabled', true)
      },
      // 禁用节点，
      invisiableNode (item) {
        this.$set(item, 'isDisabled', true)
      },
      removePerson2 () {
        let checkedNodes = this.$refs['to-tree-person'].getCheckedNodes()
        checkedNodes.forEach(item => {
          this.visiableNode(item)
          // 如果是组，全选和半选 标记 都关闭
          if (item.id === item.group_id) {
            this.halfVisiableNode(item)
            this.NotcheckedAllChildren(item.children)
          }
        })
        this.initTree()
        this.emitToParentChangeCheckedData('changeCheckedKeys',this.findAllCheckedPersonsID())
      },
      // 禁用节点，
      visiableNode (item) {
        this.$set(item, 'isDisabled', false)
      },
      // 禁用节点，
      halfVisiableNode (item) {
        this.$set(item, 'isHalfDisabled', false)
      },

      // 移除按钮

      //对树完成全选/ 全不选
      checkedOrNotAllData (tree, val) {
        this.fromData.forEach(item => {
          tree.setChecked(item, val, true)
        })
      },
      // 左侧 全选或全不选checkbox
      fromAllBoxChangePerson2 (val) {
        this.checkedOrNotAllData(this.$refs['from-tree-person'], val)
      },
      // 右侧 全选或全不选checkbox
      toAllBoxChangePerson2 (val) {
        this.checkedOrNotAllData(this.$refs['to-tree-person'], val)
      },
    }
  }
</script>

<style lang='scss' scoped>
  .root {
    display: flex;
    overflow-y: auto;
    .left,.right{
      text-align: left;
      width: 300px;
      overflow-y: auto;
    }
  }
  .left,.right{
    border: 1px solid #eeeeee;
    border-radius: 4px;
  }
  .center{
    height: 100px;
    margin: auto 20px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }
  h3{
    text-align: center;
    height:2.0em;
    line-height: 2.0em;
    padding: 0;
    margin: 0;
    color: #333;
    background: #f5f7fa;
  }
  .left-main, .right-main{
    padding: 10px;
  }
</style>
