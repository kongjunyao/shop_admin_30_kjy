<template>
  <div class="user">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索框 -->
    <el-input placeholder="请输入内容" v-model="query" class="input-with-select">
      <el-button slot="append" icon="el-icon-search" @click="searchUser"></el-button>
    </el-input>
    <!-- 用户添加按钮 -->
    <el-button type="success" plain @click="showAdd">用户添加</el-button>
    <!-- 主题表格 -->
    <el-table :data="userList" style="width: 100%">
      <el-table-column prop="username" label="姓名" width="200"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="200"></el-table-column>
      <el-table-column prop="mobile" label="电话" width="200"></el-table-column>
      <el-table-column prop="mg_state" label="状态" width="200">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
            @change="changeStatus(scope.row)"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" circle plain></el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            circle
            plain
            @click="delUser(scope.row.id)"
          ></el-button>
          <el-button type="success" icon="el-icon-check" plain>分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页器 -->
    <el-pagination
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="this.total"
    ></el-pagination>
    <!-- 添加用户模态框 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="40%">
      <!-- 表单组件 -->
      <el-form ref="addForm" status-icon :model="addForm" :rules="rules" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" placeholder="请输入用户密码"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email" placeholder="请输入用户邮箱"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="addForm.mobile" placeholder="请输入用户电话"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户模态框 -->
    <el-dialog title="添加用户" :visible.sync="editDialogVisible" width="40%">
      <!-- 表单组件 -->
      <el-form ref="addForm" status-icon :model="addForm" :rules="rules" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" placeholder="请输入用户密码"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email" placeholder="请输入用户邮箱"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="addForm.mobile" placeholder="请输入用户电话"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
// import axios from 'axios'
export default {
  data() {
    return {
      userList: [],
      query: '',
      currentPage: 1,
      total: 0,
      pageSize: 2,
      addDialogVisible: false,
      addForm: {
        username: '',
        paddword: '',
        mobile: '',
        email: ''
      },
      rules: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' },
          { min: 3, max: 9, message: '长度应该是3-9位之间', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '用户密码不能为空', trigger: 'blur' },
          { min: 6, max: 12, message: '长度应该是6-12位之间', trigger: 'blur' }
        ],
        email: [
          { type: 'email', message: '请输入一个合法的邮箱', trigger: 'blur' }
        ],
        mobile: [
          // 考虑3位
          {
            pattern: /^1[3-9]\d{9}$/,
            message: '请输入一个合法的手机号',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  methods: {
    handleSizeChange(val) {
      this.pageSize = val
      this.getUserList()
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.getUserList()
    },
    getUserList() {
      this.axios({
        methods: 'get',
        url: 'users',
        params: {
          query: this.query, // this.query 调用数据
          pagenum: this.currentPage,
          pagesize: this.pageSize
        }
      }).then(res => {
        let {
          meta: { status },
          data: { users, total }
        } = res
        if (status === 200) {
          this.userList = users
          this.total = total
        }
      })
    },
    searchUser() {
      this.currentPage = 1
      this.getUserList()
    },
    delUser(id) {
      this.$confirm('你确定删除该用户吗?', '温馨提示', {
        type: 'warning'
      }).then(() => {
        this.axios({
          url: `users/${id}`,
          method: 'delete'
        }).then(res => {
          let {
            meta: { status }
          } = res
          if (status === 200) {
            if (this.userList.length >= 1 && this.currentPage > 1) {
              this.currentPage--
            }
            this.getUserList()
            this.$message.success('恭喜你，删除成功')
          }
        })
      })
    },
    changeStatus({ id, mg_state: mgState }) {
      this.axios({
        method: 'put',
        url: 'users/' + id + '/state/' + mgState
      }).then(res => {
        let {
          meta: { status }
        } = res
        if (status === 200) {
          this.$message.success('恭喜修改成功')
        }
      })
    },
    showAdd() {
      this.addDialogVisible = true
    },
    addUser() {
      this.$refs.addForm.validate(valid => {
        if (!valid) return false
        this.axios({
          method: 'post',
          url: 'users',
          data: this.addForm
        }).then(res => {
          if (res.meta.status === 201) {
            this.$refs.addForm.resetFields()
            this.addDialogVisible = false
            this.total++
            this.currentPage = Math.ceil(this.total / this.pageSize)
            this.getUserList()
            this.$message.success('添加成功了')
          } else {
            this.$message.error(res.meta.msg)
          }
        })
      })
    }
  },
  created() {
    this.getUserList()
  }
}
</script>

<style>
.el-breadcrumb {
  height: 40px;
  line-height: 40px;
}
.input-with-select {
  width: 300px;
  margin-bottom: 5px;
}
</style>
