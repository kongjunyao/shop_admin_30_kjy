<template>
  <div class="login">
    <el-form ref="form" :model="form" :rules="rules" label-width="80px" status-icon>
      <img src="@/assets/avatar.jpg" alt>
      <el-form-item label="用户名" prop="username">
        <el-input v-model="form.username"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password" @keyup.enter.native="login">
        <el-input v-model="form.password" type="password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="login">登录</el-button>
        <el-button @click="reset">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      form: {
        username: '',
        password: ''
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
    reset() {
      this.$refs['form'].resetFields()
    },
    login() {
      this.$refs['form'].validate(valid => {
        if (!valid) return false
        axios({
          method: 'post',
          url: 'login',
          data: this.form
        }).then(res => {
          console.log(res)
          if (res.meta.status === 200) {
            this.$message({
              message: '登录成功了',
              type: 'success',
              durttion: 1000
            })
            localStorage.setItem('token', res.data.token)
            this.$router.push('/Home')
          } else {
            this.$message.error(res.meta.msg)
            // this.$message({
            //   message: this.data.meta.nsg,
            //   type: 'error'
            // })
          }
        })
      })
    }
  }
}
</script>

<style lang="less" >
.login {
  height: 100%;
  width: 100%;
  background-color: #2d434c;
  overflow: hidden;
  .el-form {
    img {
      position: absolute;
      left: 50%;
      transform: translate(-50%);
      top: -70px;
      border-radius: 50%;
      border: 10px solid #fff;
    }
    width: 400px;
    background-color: #fff;
    margin: 200px auto;
    padding: 75px 40px 15px;
    border-radius: 10px;
    position: relative;
  }
  .el-button + .el-button {
    margin-left: 80px;
  }
}
</style>
