<template>
  <div class="login">
    <el-form ref="form" :model="form" :rules="rules"  label-width="80px" status-icon>
      <img src="@/assets/avatar.jpg" alt="">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="form.username"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
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
          { required: true, message: '请输入用户名', trigger: 'change' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'change' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'change' },
          { min: 6, max: 12, message: '长度在 6 到 12 个字符', trigger: 'change' }
        ]
      }
    }
  },
  methods: {
    reset() {
      this.$refs['form'].resetFields()
    },
    login() {
      this.$refs['form'].validate((valid) => {
        if (!valid) return false
        axios({
          method: 'post',
          url: 'http://localhost:8888/api/private/v1/login',
          data: this.form
        }).then(res => {
          console.log(res.data)
          if (res.data.meta.status === 200) {
            this.$message({
              message: '登录成功了',
              type: 'success',
              durttion: 1000
            })
            localStorage.setItem('token', res.data.data.token)
            this.$router.push('/Home')
          } else {
            this.$message.error(res.data.meta.msg)
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
    img{
      position: absolute;
      left: 50%;
      transform: translate(-50%);
      top: -70px;
      border-radius: 50%;
      border: 10px solid #fff
    }
    width: 400px;
    background-color: #fff;
    margin: 200px auto;
    padding: 75px 40px 15px;
    border-radius: 10px;
    position: relative
  }
  .el-button + .el-button{
    margin-left: 80px;
  }
}
</style>
