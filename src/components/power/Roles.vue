<template>
<div>
  <!-- 面包屑导航区域 -->
  <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>权限管理</el-breadcrumb-item>
  <el-breadcrumb-item>角色列表</el-breadcrumb-item>
</el-breadcrumb>

<!-- 卡片视图 -->
<el-card>
  <!-- 添加角色按钮区 -->
  <el-row>
    <el-col>
      <el-button type="primary">添加角色</el-button>
    </el-col>
  </el-row>
  <!-- 角色列表区 -->
  <el-table :data="roleList" border stripe>
    <!-- 展开列 -->
    <el-table-column type="expand" >
      <template slot-scope="scope">
        <el-row :class="['dbbottom',i1 === 0 ? 'dbtop': '','vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
          <!-- 渲染一级权限 -->
          <el-col :span="5">
            <el-tag closable @close="removeRightsById(scope.row,item1.id)">{{item1.authName}}</el-tag>
            <i class="el-icon-caret-right"></i>
          </el-col>
          <!-- 渲染二级和三级权限 -->
          <el-col :span="19">
            <!-- 通过for循环嵌套渲染二级权限 -->
            <el-row :class="[i2 === 0 ? '': 'dbtop','vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
              <el-col :span="6">
                <el-tag type="success" closable @close="removeRightsById(scope.row,item2.id)">
                  {{item2.authName}}
                </el-tag>
                 <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="18">
                  <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable @close="removeRightsById(scope.row,item3.id)">
                    {{item3.authName}}
                  </el-tag>
              </el-col>
            </el-row>
          </el-col>
        </el-row>
      </template>
    </el-table-column>
    <!-- 索引列 -->
    <el-table-column type="index" label="#"></el-table-column>
    <el-table-column label="角色名称" prop="roleName"></el-table-column>
    <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
    <el-table-column label="操作" width="300px">
      <template slot-scope="scope">
        <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
        <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
        <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightsDialog(scope.row)">分配权限</el-button>
      </template>
    </el-table-column>
  </el-table>
</el-card>

<!-- 分配权限对话框 -->
<el-dialog
  title="分配权限"
  :visible.sync="setRightsDialog"
  width="50%" @close="setRightsDialogClosed">
  <!-- 树形控件 -->
 <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
  <span slot="footer" >
    <el-button @click="setRightsDialog = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>
</div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      roleList: [],
      // 控制分配权限对话框的显示与隐藏
      setRightsDialog: false,
      // 所有权限的数据
      rightsList: [],
      // 树形控件属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中节点ID值
      defKeys: [],
      // 即将分配权限的角色Id
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.roleList = res.data
      console.log(this.roleList)
    },
    // 根据id删除对应权限
    async removeRightsById(role, rightsId) {
      const confirmResult = await this.$confirm('此操作将取消该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (confirmResult !== 'confirm') return this.$message.info('已取消更改')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightsId}`)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      role.children = res.data
      this.$message.success(res.meta.msg)
    },
    // 展示分配权限的对话框
    async showSetRightsDialog(role) {
      // 获取所有权限的数据
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      // 把获取到的权限数据保存到data中
      this.rightsList = res.data
      console.log(this.rightsList)

      this.getLeafKeys(role, this.defKeys)
      this.setRightsDialog = true
    },
    // 通过递归的形式获取角色下所有三级权限的id并保存到 defKeys 中
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框关闭时清空
    setRightsDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      // console.log(idStr)
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.getRolesList()
      this.setRightsDialog = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag{
  margin: 7px;
}
.dbtop{
  border-top: 1px solid #eee
}
.dbbottom{
   border-bottom: 1px solid #eee
}
.vcenter{
  display: flex;
  align-items: center
}
</style>
