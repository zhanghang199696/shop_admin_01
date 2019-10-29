<template>
  <div class="users">
    <!-- 面包削 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <el-input placeholder="请输入内容" v-model="query" class="input-with-select">
      <el-button @click="searchUser" slot="append" icon="el-icon-search"></el-button>
    </el-input>
    <el-button @click="showAddDialog" class="addBtn" type="success" plain>添加用户</el-button>
    <!-- 表格 -->
    <el-table border :data="userlist">
      <el-table-column label="姓名" prop="username"></el-table-column>
      <el-table-column label="邮箱" prop="email"></el-table-column>
      <el-table-column label="电话" prop="mobile"></el-table-column>
      <el-table-column label="用户状态">
        <template v-slot:default="obj">
          <el-switch
            @change="changeUser(obj.row)"
            v-model="obj.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template v-slot:default="obj">
          <el-button
            @click="showAlterDialog(obj.row)"
            type="primary"
            plain
            icon="el-icon-edit"
            size="small"
          ></el-button>
          <el-button
            @click="del(obj.row.id)"
            type="danger"
            plain
            icon="el-icon-delete"
            size="small"
          ></el-button>
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
      :total="total"
    ></el-pagination>

    <!-- 添加模态框 -->
    <el-dialog title="添加用户" :visible.sync="dialogVisible" width="40%" @close = "closeDialog">
      <el-form :rules="rules" ref="form" :model="form" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input placeholder="请输入用户名" v-model="form.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input type="password" placeholder="请输入密码" v-model="form.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input placeholder="请输入邮箱" v-model="form.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input placeholder="请输入手机" v-model="form.mobile"></el-input>
        </el-form-item>
      </el-form>

      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button @click="addUsers" type="primary">确 定</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 修改用户 -->
    <el-dialog title="添加用户" :visible.sync="editVisible" width="40%">
      <el-form :rules="rules" ref="editform" :model="editForm" label-width="80px">
        <el-form-item type="info" label="用户名">{{editForm.username}}</el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input placeholder="请输入邮箱" v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input placeholder="请输入手机" v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="editVisible = false">取 消</el-button>
          <el-button @click="alterUser" type="primary">确 定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
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
      total: 0,
      dialogVisible: false,
      editVisible: false,
      form: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      rules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: ['blur', 'change']
          },
          {
            min: 3,
            max: 12,
            message: '长度在 3 到 12 个字符',
            trigger: ['blur', 'change']
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: ['blur', 'change']
          },
          {
            min: 3,
            max: 12,
            message: '长度在 3 到 12 个字符',
            trigger: ['blur', 'change']
          }
        ],
        email: [
          {
            type: 'email',
            message: '请输入正确的邮箱',
            trigger: ['blur', 'change']
          }
        ],
        mobile: [
          {
            pattern: /^1[3-9]\d{9}$/,
            message: '请输入正确手机号',
            trigger: ['blur', 'change']
          }
        ]
      },
      editForm: {
        email: '',
        mobile: '',
        username: '',
        id: ''
      }
    }
  },
  methods: {
    async getUserList () {
      const { meta, data } = await this.$axios.get('users', {
        params: {
          query: this.query,
          pagenum: this.pagenum,
          pagesize: this.pagesize
        }
      })
      if (meta.status === 200) {
        this.userlist = data.users
        this.total = data.total
      } else {
        this.$message.error(meta.msg)
      }
    },
    // 处理每页条数的变化
    handleSizeChange (val) {
      // console.log(val)
      this.pagesize = val
    },
    // 处理当前页的变化
    handleCurrentChange (val) {
      this.pagenum = val
      this.getUserList()
    },
    async del (id) {
      try {
        await this.$confirm('亲!您确认要删除吗', '温馨提示', {
          type: 'warning'
        })
        // 发送axios的请求
        const { meta } = await this.$axios.delete(`users/${id}`)
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          if (this.userlist.length === 1 && this.pagenum > 1) {
            this.pagenum--
          }
          this.getUserList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    searchUser () {
      this.pagenum = 1
      this.getUserList()
    },
    // 改变用户的状态
    async changeUser (row) {
      const { meta } = await this.$axios.put(
        `users/${row.id}/state/${row.mg_state}`
      )
      if (meta.status === 200) {
        this.$message.success(meta.msg)
      } else {
        this.$message.error(meta.msg)
      }
    },
    showAddDialog () {
      this.dialogVisible = true
    },
    async addUsers () {
      try {
        await this.$refs.form.validate()
        const { meta } = await this.$axios.post('users', this.form)
        if (meta.status === 201) {
          this.dialogVisible = false
          this.$message.success(meta.msg)
          this.total++
          this.pagenum = Math.ceil(this.total / this.pagesize)
          this.getUserList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    showAlterDialog (row) {
      this.editVisible = true
      this.editForm.username = row.username
      this.editForm.id = row.id
      this.editForm.email = row.email
      this.editForm.mobile = row.mobile
    },
    closeDialog () {
      this.$refs.form.resetFields()
    },
    async alterUser () {
      try {
        await this.$refs.editform.validate()
        const { id, email, mobile } = this.editForm
        const { meta } = await this.$axios.put(`users/${id}`, { email, mobile })
        // console.log(res)

        if (meta.status === 200) {
          this.$message.success(meta.msg)
          this.editVisible = false
          this.getUserList()
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

<style lang='scss' scoped>
.users {
  .el-breadcrumb {
    border-bottom: 1px solid #ccc;
    height: 40px;
    line-height: 40px;
    margin-bottom: 20px;
  }
  .input-with-select {
    width: 300px;
    margin-bottom: 20px;
    margin-right: 15px;
  }
}
</style>
