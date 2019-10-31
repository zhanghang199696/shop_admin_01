<template>
  <div class="categories">
    <!-- 添加按钮 -->
    <el-button @click="showAddDialog" type="success" plain>添加分类</el-button>
    <!-- 表格部分 -->
    <el-table
      :data="categoriesList"
      row-key="cat_id"
      v-loading="isLoading"
      element-loading-text="拼命加载中..."
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(255, 255, 255, 0.9)"
    >
      <el-table-column label="分类名称" prop="cat_name"></el-table-column>
      <el-table-column label="是否有效">
        <template v-slot:default="{row}">{{ row.cat_deleted ? '否' : '是' }}</template>
      </el-table-column>
      <el-table-column label="排序" prop="cat_level"></el-table-column>
      <el-table-column label="操作">
        <template v-slot:default="{row}">
          <el-button @click="showDailog(row)" type="primary" plain icon="el-icon-edit" size="small"></el-button>
          <el-button @click="Del(row)" type="danger" plain icon="el-icon-delete" size="small"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 编辑模态框 -->
    <el-dialog title="提示" :visible.sync="dialogVisible" width="40%">
      <el-form :model="form" label-width="80px">
        <el-form-item label="分类名称">
          <el-input v-model="form.catName"></el-input>
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="category">确 定</el-button>
        </span>
      </template>
    </el-dialog>
    <!-- 添加模态框 -->
    <el-dialog title="提示" :visible.sync="addDialog" width="40%" @close="closeDialog">
      <el-form :rules="rules" ref="form" :model="addForm" label-width="80px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addForm.cat_name" placeholder="请输入分类名称"></el-input>
        </el-form-item>
        <el-form-item label="父级名称">
          <el-cascader v-model="addForm.cat_pid" :options="options" :props="props"></el-cascader>
        </el-form-item>
      </el-form>
      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="addDialog = false">取 消</el-button>
          <el-button @click="addCategories" type="primary">确 定</el-button>
        </span>
      </template>
    </el-dialog>
    <!-- 分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[5, 10, 15]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>
  </div>
</template>

<script>
export default {
  data () {
    return {
      categoriesList: [],
      pagenum: 1,
      pagesize: 5,
      total: 0,
      isLoading: false,
      dialogVisible: false,
      addDialog: false,
      options: [],
      props: {
        checkStrictly: true,
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      form: {
        id: '',
        catName: ''
      },
      addForm: {
        cat_pid: [],
        cat_name: '',
        cat_level: ''
      },
      rules: {
        cat_name: [
          {
            required: true,
            message: '分类名称不能为空',
            trigger: ['blur', 'change']
          }
        ]
      }
    }
  },
  created () {
    this.getCategoriesList()
  },
  methods: {
    async getCategoriesList () {
      this.isLoading = true
      const { meta, data } = await this.$axios.get('categories', {
        params: {
          pagenum: this.pagenum,
          pagesize: this.pagesize,
          type: 3
        }
      })
      if (meta.status === 200) {
        this.categoriesList = data.result
        this.total = data.total
        this.isLoading = false
      } else {
        this.$message.error(meta.msg)
      }
    },
    handleSizeChange (val) {
      this.pagesize = val
      this.pagenum = 1
      this.getCategoriesList()
    },
    handleCurrentChange (val) {
      this.pagenum = val
      this.getCategoriesList()
    },
    async Del (row) {
      try {
        console.log(row)

        await this.$confirm('你确定要删除吗', '温馨提示')
        const { meta } = await this.$axios.delete(`categories/${row.cat_id}`)
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          this.getCategoriesList()
        } else {
          this.$message.console.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    showDailog (row) {
      console.log(row)
      this.form.id = row.cat_id
      this.form.catName = row.cat_name
      this.dialogVisible = true
    },
    async category () {
      const { id, catName } = this.form
      const { meta } = await this.$axios.put(`categories/${id}`, {
        cat_name: catName
      })
      if (meta.status === 200) {
        this.$message.success(meta.msg)
        this.dialogVisible = false
        this.getCategoriesList()
      } else {
        this.$message.error(meta.msg)
        this.dialogVisible = false
      }
    },
    async showAddDialog () {
      this.addDialog = true
      // 发送axios
      const { meta, data } = await this.$axios.get('categories?type=2')
      if (meta.status === 200) {
        this.options = data
        console.log(data)
      } else {
        this.$message.error(meta.msg)
      }
    },
    async addCategories () {
      try {
        await this.$refs.form.validate()
        const { meta } = await this.$axios.post('categories', {
          cat_pid: this.addForm.cat_pid[this.addForm.cat_pid.length - 1] || 0,
          cat_name: this.addForm.cat_name,
          cat_level: this.addForm.cat_pid.length
        })
        if (meta.status === 201) {
          this.$message.success(meta.msg)
          this.addDialog = false
          this.getCategoriesList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    closeDialog () {
      this.$refs.form.resetFields()
    }
  }
}
</script>

<style lang="scss" scoped>
.categories {
  .el-button {
    margin-bottom: 10px;
  }
}
</style>
