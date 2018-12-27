<template>
  <div class="user">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-input placeholder="请输入内容" v-model="query">
      <el-button slot="append" icon="el-icon-search" @click="searchUser"></el-button>
    </el-input>
    <el-button type="success" plain>用户添加</el-button>
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
      baseURL: 'http://localhost:8888/api/private/v1/'
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
        url: this.baseURL + 'users',
        params: {
          query: this.query, // this.query 调用数据
          pagenum: this.currentPage,
          pagesize: this.pageSize
        },
        headers: {
          Authorization: localStorage.getItem('token')
        }
      }).then(res => {
        console.log(res.data)
        if (res.data.meta.status === 200) {
          this.userList = res.data.data.users
          this.total = res.data.data.total
          console.log(this.userList)
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
          url: this.baseURL + `users/${id}`,
          method: 'delete',
          headers: {
            Authorization: localStorage.getItem('token')
          }
        }).then(res => {
          if (res.data.meta.status === 200) {
            if (this.userList.length >= 1 && this.currentPage > 1) {
              this.currentPage--
            }
            this.getUserList()
            this.$messgae.success('恭喜你，删除成功')
          }
        })
      })
    },
    changeStatus(user) {
      this.axios({
        method: 'put',
        url: this.baseURL + 'users/' + user.id + '/state/' + user.mg_state,
        headers: {
          Authorization: localStorage.getItem('token')
        }
      }).then(res => {
        if (res.data.meta.status === 200) {
          this.$message.success('恭喜修改成功')
        }
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
.el-input {
  width: 300px;
  margin-bottom: 5px;
}
</style>
