<template>
  <div class="users">

    <el-table border :data="userlist">
      <el-table-column label="姓名" prop="username"></el-table-column>
      <el-table-column label="邮箱" prop="email"></el-table-column>
      <el-table-column label="电话" prop="mobile"></el-table-column>
      <el-table-column label="用户状态">
        <template v-slot:default="obj">
          <el-switch v-model="obj.row.mg_state" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template>
          <el-button type="primary" plain icon="el-icon-edit" size="small"></el-button>
          <el-button type="danger" plain icon="el-icon-delete" size="small"></el-button>
          <el-button type="success" plain icon="el-icon-check" size="small">分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
     @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8,10]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  created () {
    this.getUserList()
  },
  data () {
    return {
      userlist: [],
      query: '',
      pagenum: 1, // 当前页
      pagesize: 2, // 每页条数
      total: 0
    }
  },
  methods: {
    getUserList () {
      axios
        .get('http://localhost:8888/api/private/v1/users', {
          params: {
            query: this.query,
            pagenum: this.pagenum,
            pagesize: this.pagesize
          },
          headers: {
            Authorization: localStorage.getItem('token')
          }
        })
        .then(res => {
          const { data, meta } = res.data

          if (meta.status === 200) {
            this.userlist = data.users
            this.total = data.total
            // console.log(res.data)
          } else {
            this.$message.error(meta.msg)
          }
        })
    },
    // 处理每页条数的变化
    handleSizeChange (val) {
      // console.log(val)
      this.pagesize = val
    },
    // 处理当前页的变化
    handleCurrentChange (val) {
      console.log(val)
      this.pagenum = val
      this.getUserList()
    }
  }
}
</script>

<style>
</style>
