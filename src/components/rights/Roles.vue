<template>
  <div class="roles">
    <!-- 面包削 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 添加角色按钮 -->
    <el-button @click="showAddDialog" class="addRole" type="success" plain>添加角色</el-button>
    <!-- 表格 -->
    <el-table :data="rolesList">
      <el-table-column type="expand">
        <template v-slot:default="{row}">
          <el-row class="l1" v-for="l1 in row.children" :key="l1.id">
            <el-col :span="4">
              <el-tag @close="delRoles(row,l1.id)" closable>{{l1.authName}}</el-tag>
              <i class="el-icon-arrow-right"></i>
            </el-col>
            <el-col :span="20">
              <!-- 每个二级都是一行 -->
              <el-row class="l2" v-for="l2 in l1.children" :key="l2.id">
                <el-col :span="4">
                  <el-tag @close="delRoles(row,l2.id)" closable type="success">{{l2.authName}}</el-tag>
                  <i class="el-icon-arrow-right"></i>
                </el-col>
                <el-col :span="20">
                  <el-tag
                    closable
                    type="warning"
                    class="l3"
                    v-for="l3 in l2.children"
                    :key="l3.id"
                    @close="delRoles(row,l3.id)"
                  >{{l3.authName}}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column type="index"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作">
        <template v-slot:default="{row}">
          <el-button
            @click="showModificationDialog(row)"
            type="primary"
            plain
            icon="el-icon-edit"
            size="small"
          ></el-button>
          <el-button @click="delRoles2(row)" type="danger" plain icon="el-icon-delete" size="small"></el-button>
          <el-button
            @click="showApportionDialog(row)"
            type="success"
            plain
            icon="el-icon-check"
            size="small"
          >分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分配角色模态框 -->
    <el-dialog :title="title" :visible.sync="showDialog" width="40%">
      <el-tree
        ref="tree"
        :data="data"
        :props="defaultProps"
        show-checkbox
        default-expand-all
        node-key="id"
      ></el-tree>
      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="showDialog = false">取 消</el-button>
          <el-button type="primary" @click="assignRight">分配</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 修改与添加的公用模态框 -->
    <el-dialog @close="closeDialog" :title="title" :visible.sync="saveShow" width="40%">
      <el-form :rules="rules" ref="form" :model="form" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="form.roleName" placeholder="请输入角色名称"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="form.roleDesc" placeholder="请输入角色描述"></el-input>
        </el-form-item>
      </el-form>

      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="saveShow = false">取 消</el-button>
          <el-button type="primary" @click="saveUser">确定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      roleName: '',
      roleDesc: '',
      id: '',
      showDialog: false,
      data: [],
      saveShow: false,
      title: '',
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      rowId: '',
      form: {
        id: '',
        roleName: '',
        roleDesc: ''
      },
      rules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: ['blur', 'change'] }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: ['blur', 'change'] }
        ]
      }
    }
  },
  async created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data, meta } = await this.$axios.get('roles')
      if (meta.status === 200) {
        this.rolesList = data
      } else {
        this.$message.error(meta.msg)
      }
    },
    async delRoles (row, id) {
      const { meta, data } = await this.$axios.delete(
        `roles/${row.id}/rights/${id}`
      )
      if (meta.status === 200) {
        this.$message.success(meta.msg)
        row.children = data
      } else {
        this.$message.errey(meta.msg)
      }
    },
    async showApportionDialog (row) {
      this.rowId = row.id
      this.showDialog = true
      const { meta, data } = await this.$axios.get('rights/tree')
      if (meta.status === 200) {
        this.data = data
        const arr = []
        row.children.forEach(l1 => {
          l1.children.forEach(l2 => {
            l2.children.forEach(l3 => {
              arr.push(l3.id)
            })
          })
        })
        this.$refs.tree.setCheckedKeys(arr)
        this.getRolesList()
      }
    },
    async assignRight () {
      // 获取选中的权限id (全选中的 和 半选中的)
      const ids = this.$refs.tree.getCheckedKeys()
      const halfs = this.$refs.tree.getHalfCheckedKeys()
      const rids = [...ids, ...halfs].join(',')
      const { meta } = await this.$axios.post(`roles/${this.rowId}/rights`, {
        rids
      })
      if (meta.status === 200) {
        this.$message.success('分配成功')
        this.showDialog = false
        this.getRolesList()
      } else {
        this.$message.error(meta.msg)
      }
    },
    async delRoles2 (row) {
      try {
        await this.$confirm('您确认要删除吗?', '温馨提示')
        const { meta } = await this.$axios.delete(`roles/${row.id}`)
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          this.getRolesList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
      // await this.$config('您确认要删除吗?', '温馨提示')
      // const { meta } = await this.$axios.delete(`roles/${row.id}`)
      // if (meta.status === 200) {
      //   this.$message.success(meta.msg)
      // }
    },
    showAddDialog () {
      this.title = '添加角色'
      this.saveShow = true
    },
    showModificationDialog (row) {
      this.form.id = ''
      this.title = '修改角色'
      this.saveShow = true
      // 先显示再改值
      this.$nextTick(() => {
        this.form.id = row.id
        this.form.roleName = row.roleName
        this.form.roleDesc = row.roleDesc
      })
    },
    // 关闭模态框重置内容
    closeDialog () {
      this.$refs.form.resetFields()
    },
    async saveUser () {
      try {
        // 先校验
        await this.$refs.form.validate()

        const { id } = this.form
        console.log(id)

        let res
        if (id) {
          res = await this.$axios.put(`roles/${id}`, this.form)
        } else {
          res = await this.$axios.post('roles', this.form)
        }
        const { meta } = res
        console.log(meta)

        if (meta.status === 200 || meta.status === 201) {
          this.$message.success(meta.msg)
          this.saveShow = false
          this.getRolesList()
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

<style lang="scss" scoped>
.roles {
  .l1 {
    padding: 10px 0;
    border-bottom: 1px dotted #ccc;
    &:last-child {
      border-bottom: none;
    }
  }
  .addRole {
    margin-bottom: 10px;
  }
  .l2 {
    margin-bottom: 5px;
  }
  .l3 {
    margin-right: 5px;
    margin-bottom: 5px;
  }
}
</style>
