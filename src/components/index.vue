<template>
  <el-container class="index">
    <el-header>
      <div class="logo">
        <img src="../assets/logo.png" alt />
      </div>
      <div class="main">电商后台管理系统</div>
      <div class="logout">
        欢迎光临~
        <a href="javascript:;" @click="logout">退出</a>
      </div>
    </el-header>
    <el-container>
      <el-aside width="200px">
        <!-- 整个容器 -->
        <el-menu
          router
          unique-opened
          :default-active="defaultActive"
          class="el-menu-vertical-demo"
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b"
        >
          <!-- 子菜单 -->
          <el-submenu :index="menu.path" v-for="menu in menuList" :key="menu.id">
            <!-- 标题 -->
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>{{menu.authName}}</span>
            </template>
            <!-- 内容 -->
            <el-menu-item :index="item.path" v-for="item in menu.children" :key="item.id">
              <i class="el-icon-menu"></i>
              <span slot="title">{{item.authName}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  computed: {
    defaultActive () {
      return this.$route.path.slice(1).split('A')[0]
    }
  },
  data () {
    return {
      menuList: []
    }
  },
  async created () {
    const { meta, data } = await this.$axios.get('menus')
    if (meta.status === 200) {
      this.menuList = data
    } else {
      this.$message.error(meta.msg)
    }
  },
  methods: {
    async logout () {
      try {
        const res = await this.$confirm(
          '尊敬的用户,您确认要推出吗？',
          '温馨提示',
          {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }
        )
        if (res) {
          localStorage.removeItem('token')
          this.$router.push('/login')
          this.$message({
            type: 'success',
            message: '退出成功!'
          })
        }
      } catch (e) {
        console.log(e)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
* {
  padding: 0;
  margin: 0;
}
.index {
  height: 100%;
  .el-header {
    display: flex;
    background: #d8d8d8;
    line-height: 60px;
    padding: 0 20px;
    img {
      height: 40px;
      margin-top: 10px;
    }
    .main {
      font-weight: 700;
      color: #545c64;
      flex: 1;
      text-align: center;
      font-size: 24px;
    }
    .logout {
      font-weight: 600;
    }
    a {
      color: orange;
    }
  }
  .el-aside {
    background: #545c64;
    .el-menu {
      border: 0;
    }
  }
  .el-main {
    background: #ecf0f1;
  }
}
</style>
