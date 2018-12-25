<template>

<el-container class="home">
  <el-header>
    <div class="logo"></div>
    <div class="title">电商后台管理系统</div>
    <div class="logout">欢迎光临<a href="javascript:;" @click="logout">退出</a></div>
  </el-header>
  <el-container>
    <el-aside width="200px">Aside</el-aside>
    <el-main>Main</el-main>
  </el-container>
</el-container>

</template>

<script>
export default {
  methods: {
    logout () {
      localStorage.removeItem('token')
      this.$router.push('/login')
      this.$message.success('退出系统成功')
    }
  }
}
</script>

<style lang="less" scoped>
 .home{
   height: 100%;
   .el-header{
      height: 60px;
      background-color: #B3C1CD;
      display: flex;
      .logo{
        width: 180px;
        background-image: url('~@/assets/logo.png');
        background-size: contain;
        background-repeat: no-repeat;
        background-position: center center;
      }
      .logout{
        width: 180px;
        line-height: 60px;
        font-weight: 700;
        text-align: right;
        a{
          color: orange;
        }
      }
      .title{
        flex: 1;
        line-height: 60px;
        text-align: center;
        font-size: 30px;
        color: #fff;
        font-weight: 700;
      }
   }
   .el-aside{
      background-color: #545C64;
   }
   .el-main{
      background-color: #D4DFE4;
   }
 }
</style>
