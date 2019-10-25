<template>
  <el-container class="index">
  <el-header>
    <div class="logo">
      <img src="../assets/logo.png" alt="">
    </div>
    <div class="main">电商后台管理系统</div>
    <div class="logout">
      欢迎光临~<a href="javascript:;" @click="logout">退出</a>
    </div>
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
      this.$confirm('尊敬的用户,您确认要推出吗？', '温馨提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        localStorage.removeItem('token')
        this.$router.push('/login')
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
      }).catch(() => {
        console.log()
      })
    }
  }
}
</script>

<style lang="scss" scoped>
*{
  padding: 0;
  margin: 0;
}
.index{
  height: 100%;
  .el-header{
    display: flex;
    background: #d8d8d8;
    line-height: 60px;
    padding: 0 20px;
    img{
      height: 40px;
      margin-top: 10px;
    }
    .main{
      font-weight: 700;
      color: #545c64;
      flex: 1;
      text-align: center;
      font-size: 24px;
    }
    .logout{
      font-weight: 600;
    }
    a{
      color: orange;
    }
  }
}

</style>
