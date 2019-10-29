<template>
  <div class="login">
    <el-form ref="form" :rules="rules" :model="form" label-width="80px">
      <img src="../assets/01.jpg" alt />
      <el-form-item label="用户名" prop="username">
        <el-input placeholder="请输入用户名" v-model="form.username"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input type="password" placeholder="密码" v-model="form.password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="login">登录</el-button>
        <el-button @click="result">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
// import axios from 'axios'
export default {
  data () {
    return {
      form: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' },
          {
            min: 3,
            max: 12,
            message: '用户名的长度必须在3-12位之间',
            trigger: ['blue', 'change']
          }
        ],
        password: [
          { required: true, message: '密码', trigger: 'blur' },
          {
            min: 3,
            max: 12,
            message: '密码的长度必须在3-12位之间',
            trigger: ['blue', 'change']
          }
        ]
      }
    }
  },
  methods: {
    result () {
      this.$refs.form.resetFields()
    },
    // 点击登录去首页
    async login () {
      try {
        await this.$refs.form.validate()
        const { meta, data } = await this.$axios.post('login', this.form)
        if (meta.status === 200) {
          localStorage.setItem('token', data.token)
          this.$message({
            message: meta.msg,
            type: 'success',
            duration: 1000
          })
          this.$router.push('/index')
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    }
  }
}
</script>
<style lang='scss'>
.login {
  width: 100;
  height: 100%;
  background: #2d434c;
  overflow: hidden;
  .el-form {
    width: 400px;
    padding: 20px;
    padding-top: 70px;
    border-radius: 20px;
    background: #ffffff;
    margin: 0 auto;
    margin-top: 200px;
    position: relative;
    .el-button:first-child {
      margin-right: 80px;
    }
    img {
      width: 120px;
      height: 120px;
      position: absolute;
      border-radius: 50%;
      border: 5px solid #fff;
      left: 50%;
      top: -60px;
      transform: translateX(-50%);
    }
  }
}
</style>
