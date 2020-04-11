<template>
  <div>
    <el-tree :data="menus" :props="defaultProps"
             show-checkbox
             node-key="catId"
             :default-expanded-keys="expandedNode"
             :draggable="true"
             :allow-drop="allowDrop"
             :expand-on-click-node="false">
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-show="node.level<=2"
            type="text"
            size="mini"
            @click="() => append(data)">
            新增
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)">
            编辑
          </el-button>
          <el-button
            v-show="node.childNodes.length===0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            删除
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :close-on-click-modal="false"
      :visible.sync="dialogVisible">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
      <el-button @click="dialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="submit">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        title: '',
        maxLevel: 0,
        submitType: '',
        category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, productUnit: null, icon: ''},
        dialogVisible: false,
        menus: [],
        // 默认展开的节点
        expandedNode: [],
        defaultProps: {
          children: 'children',
          label: 'name'
        }
      }
    },
    methods: {
      allowDrop (draggingNode, dropNode, type) {
        // 判断被拖动当前节点以及所在父节点总层级是否大于3
        console.log(draggingNode, dropNode, type)
        // 计算当前节点最大层级
        this.calculationNodeLevel(draggingNode.data)
        let deep = Math.abs(this.maxLevel - draggingNode.data.catLevel + 1)
        console.log('深度', deep)
        if (type === 'inner') {
          return (deep + dropNode.level) <= 3
        } else {
          return (deep + dropNode.parent.level) <= 3
        }
      },
      calculationNodeLevel (node) {
        if (node.children != null && node.children.length > 0) {
          for (let i = 0; i < node.children.length; i++) {
            if (node.children[i].catLevel > this.maxLevel) {
              this.maxLevel = node.children[i].catLevel
            }
            this.calculationNodeLevel(node.children[i])
          }
        }
      },
      submit () {
        if (this.submitType === 'add') {
          this.addCategory()
        }
        if (this.submitType === 'edit') {
          this.editCategory()
        }
      },
      edit (data) {
        this.dialogVisible = true
        this.submitType = 'edit'
        this.title = '修改分类'
        // 发送请求获取最新的数据
        this.$http({
          url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
          method: 'get'
        }).then(({data}) => {
          let result = data.data
          this.category.name = result.name
          this.category.catId = result.catId
          this.category.parentCid = result.parentCid
          this.category.catLevel = result.catLevel
          this.category.productUnit = result.productUnit
          this.category.icon = result.icon
        }).catch(error => {
          console.log(error)
        })
      },
      editCategory () {
        this.dialogVisible = false
        let {catId, name, productUnit, icon} = this.category
        this.$http({
          url: this.$http.adornUrl('/product/category/update'),
          method: 'post',
          data: this.$http.adornData({catId, name, productUnit, icon}, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success'
            })
            this.getMenuData()
            // 设置默认展开的菜单
            this.expandedNode = [this.category.parentCid]
            this.category = {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, productUnit: null, icon: ''}
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      addCategory () {
        this.dialogVisible = false
        this.$http({
          url: this.$http.adornUrl('/product/category/save'),
          method: 'post',
          data: this.$http.adornData(this.category, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success'
            })
            this.getMenuData()
            // 设置默认展开的菜单
            this.expandedNode = [this.category.parentCid]
            this.category = {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, productUnit: null, icon: ''}
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      append (data) {
        this.dialogVisible = true
        this.title = '新增分类'
        this.submitType = 'add'
        this.category.parentCid = data.catId
        this.category.icon = data.icon
        this.category.productUnit = data.productUnit
        this.category.catLevel = data.catLevel + 1
      },

      remove (node, data) {
        this.$confirm(`确定删除【${data.name}】菜单？`, {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let ids = [data.catId]
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success'
              })
              this.getMenuData()
              // 设置默认展开的菜单
              this.expandedNode = [node.parent.data.catId]
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {
        })
      },
      getMenuData () {
        this.$http({
          url: this.$http.adornUrl('/product/category/getCategory/tree'),
          method: 'get'
        }).then(({data}) => {
          this.menus = data.data
        }).catch(error => {
          console.log(error)
        })
      }
    },
    created () {
      this.getMenuData()
    }
  }
</script>

<style>

</style>
