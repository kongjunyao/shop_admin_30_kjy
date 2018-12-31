<template>
  <div class="roles">
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 添加按钮 -->
    <el-button type="success" plain @click="showAddRole">添加角色</el-button>
    <!-- 表格 -->
    <el-table :data="roleList">
      <el-table-column type="expand">
        <template slot-scope="{row}">
          <el-row class="l1" v-for="l1 in row.children" :key="l1.id">
            <el-col :span="4">
              <el-tag closable @close="delRole(row, l1.id)">{{l1.authName}}</el-tag>
              <i class="el-icon-arrow-right"></i>
            </el-col>
            <el-col :span="20">
              <el-row class="l2" v-for="l2 in l1.children" :key="l2.id">
                <el-col :span="4">
                  <el-tag closable @close="delRole(row, l2.id)">{{l2.authName}}</el-tag>
                  <i class="el-icon-arrow-right"></i>
                </el-col>
                <el-col :span="20">
                  <el-tag
                    @close="delRight(row, l3.id)"
                    closable
                    class="l3"
                    type="danger"
                    v-for="l3 in l2.children"
                    :key="l3.id"
                  >{{l3.authName}}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column type="index"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="{row}">
          <el-button
            type="primary"
            plain
            icon="el-icon-edit"
            circle
            size="mini"
            @click="showEdit(row)"
          ></el-button>
          <el-button
            type="danger"
            plain
            icon="el-icon-delete"
            circle
            size="mini"
            @click="delRoles(row.id)"
          ></el-button>
          <el-button
            type="success"
            plain
            icon="el-icon-check"
            round
            size="mini"
            @click="showAssign(row)"
          >分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="分配权限" :visible.sync="assignDialogVisible" width="40%">
      <el-tree
        :data="rightList"
        :props="defaultProps"
        show-checkbox
        node-key="id"
        default-expand-all
        ref="tree"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="assignDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="assignRole">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="40%">
      <el-form ref="addForm" status-icon :model="addForm" :rules="rules" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName" placeholder="请输入角色名"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改 -->
    <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="40%">
      <el-form ref="addForm" status-icon :model="editForm" :rules="rules" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName" placeholder="请输入角色名"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="updataRole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roleList: [],
      assignDialogVisible: false,
      addDialogVisible: false,
      editDialogVisible: false,
      rightList: [],
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      roleId: '',
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      editForm: {
        id: '',
        roleName: '',
        roleDesc: ''
      },
      rules: {
        roleName: [
          { required: true, message: '用户名不能为空', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    async getRoleList() {
      let res = await this.axios.get('roles')
      console.log(res)
      if (res.meta.status === 200) {
        this.roleList = res.data
      }
    },
    async delRole(role, rightId) {
      let res = await this.axios.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status === 200) {
        // 删除成功
        // this.getRoleList()
        // 删除成功后，不能重新渲染整个角色列表，只重新渲染了当前角色的权限信息
        role.children = res.data
        // 提示消息
        this.$message.success('删除权限成功了')
        // console.log(res)
      }
    },
    async showAssign(role) {
      this.roleId = role.id
      this.assignDialogVisible = true
      let res = await this.axios.get('rights/tree')
      if (res.meta.status === 200) {
        this.rightList = res.data
      }
      let ids = []
      role.children.forEach(l1 => {
        l1.children.forEach(l2 => {
          l2.children.forEach(l3 => {
            ids.push(l3.id)
          })
        })
      })
      this.$refs.tree.setCheckedKeys(ids)
    },
    async assignRole() {
      let setCheckedKeys = this.$refs.tree.getCheckedKeys()
      let halfCheckdKeys = this.$refs.tree.getHalfCheckedKeys()
      let rids = setCheckedKeys.concat(halfCheckdKeys).join()
      let res = await this.axios.post(`roles/${this.roleId}/rights`, { rids })
      if (res.meta.status === 200) {
        this.assignDialogVisible = false
        this.getRoleList()
        this.$message.success('分配权限成功了')
      }
    },
    async delRoles(id) {
      try {
        await this.$confirm('你确定要删除该角色吗', '温馨提示', {
          type: 'warning'
        })
        let res = await this.axios.delete(`roles/${id}`)
        if (res.meta.status === 200) {
          this.getRoleList()
          this.$message.success('删除成功')
        }
      } catch (e) {}
    },
    showAddRole() {
      this.addDialogVisible = true
    },
    async addRole() {
      try {
        await this.$refs.addForm.validate()
        let res = await this.axios({
          method: 'post',
          url: 'roles',
          data: this.addForm
        })
        console.log(res)
        if (res.meta.status === 201) {
          this.addDialogVisible = false
          this.$refs.addForm.resetFields()
          this.getRoleList()
          this.$message.success('添加成功了')
        } else {
          this.$message.error(res.meta.msg)
        }
      } catch (e) {
        return false
      }
    },
    showEdit(user) {
      this.editDialogVisible = true
      this.editForm.id = user.id
      this.editForm.roleName = user.roleName
      this.editForm.roleDesc = user.roleDesc
    },
    async updataRole() {
      try {
        let id = this.editForm.id
        let res = await this.axios({
          method: 'put',
          url: `roles/${id}`,
          data: this.editForm
        })
        console.log(res)
        if (res.meta.status === 200) {
          this.editDialogVisible = false
          this.getRoleList()
          this.$message.success('操作成功了')
        }
      } catch (e) {
        return false
      }
    }
  },
  created() {
    this.getRoleList()
  }
}
</script>

<style>
.el-button {
  margin-top: 10px;
}
.l1 {
  padding: 5px 0;
}

.l3 {
  margin-right: 5px;
  margin-bottom: 5px;
}
</style>
