<template>
  <div class="GoodsAdd">
    <!-- 步骤条 -->
    <el-steps :active="active" finish-status="success">
      <el-step title="基本信息"></el-step>
      <el-step title="商品图片"></el-step>
      <el-step title="商品内容"></el-step>
    </el-steps>
    <!-- tab栏 -->
    <el-tabs v-model="activeName" @tab-click="handleClick" tab-position="left">
      <el-tab-pane name="0" label="基本信息">
        <el-form ref="form" :model="form" label-width="80px">
          <el-form-item label="商品名称">
            <el-input v-model="form.goods_name"></el-input>
          </el-form-item>
          <el-form-item label="商品价格">
            <el-input v-model="form.goods_price"></el-input>
          </el-form-item>
          <el-form-item label="商品重量">
            <el-input v-model="form.goods_weight"></el-input>
          </el-form-item>
          <el-form-item label="商品数量">
            <el-input v-model="form.goods_number"></el-input>
          </el-form-item>
          <el-form-item label="商品分类">
            <!-- 级联 -->
            <el-cascader v-model="form.goods_cat" :options="options" :props="props"></el-cascader>
          </el-form-item>
        </el-form>
        <el-button @click="goNext" type="primary">下一步</el-button>
      </el-tab-pane>
      <el-tab-pane name="1" label="商品图片">
        <el-upload
          action="http://localhost:8888/api/private/v1/upload"
          list-type="picture-card"
          name="file"
          multiple
          :headers="headers"
          :on-success="handleSuccess"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-button @click="goNext2" type="primary">下一步</el-button>
      </el-tab-pane>
      <el-tab-pane name="2" label="商品内容">
        <quill-editor v-model="form.goods_introduce"></quill-editor>
        <el-button @click="addGood" type="primary">添加</el-button>
      </el-tab-pane>
    </el-tabs>
    <el-dialog :visible.sync="dialogVisible">
      <img width="100%" :src="dialogImageUrl" alt />
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      active: 0,
      activeName: '0',
      dialogVisible: false,
      dialogImageUrl: '',
      form: {
        goods_name: '', // 商品名称
        goods_cat: '', // 以为','分割的分类列表 (字符串)
        goods_price: '', // 价格
        goods_number: '', // 数量
        goods_weight: '', // 重量
        goods_introduce: '', // 介绍
        pics: [] // 数组, 存着对象（上传的图片临时路径）
      },
      options: [],
      props: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      headers: {
        Authorization: localStorage.getItem('token')
      }
    }
  },
  async created () {
    const { data, meta } = await this.$axios.get('categories?type=3')
    if (meta.status === 200) {
      this.options = data
    } else {
      this.$message.error(meta.msg)
    }
  },
  methods: {
    handleClick (tab) {
      this.active = +tab.index
    },
    goNext () {
      this.active++
      this.activeName = this.active.toString()
    },
    handleSuccess (response, file, fileList) {
      const { meta, data } = response
      if (meta.status === 200) {
        this.form.pics.push({
          pic: data.tmp_path
        })
      } else {
        this.$message.error(meta.msg)
      }
    },
    // 删除
    handleRemove (file, fileList) {
      const path = file.response.data.tmp_path
      this.form.pics = this.form.pics.filter(item => item.pic !== path)
    },
    handlePictureCardPreview (file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    },
    goNext2 () {
      this.active++
      this.activeName = this.active.toString()
    },
    async addGood () {
      const { meta } = await this.$axios.post('goods', {
        ...this.form,
        goods_cat: this.form.goods_cat.join(',')
      })
      if (meta.status === 201) {
        this.$message.success(meta.msg)
        this.$router.push('/goods')
      } else {
        this.$message.error(meta.msg)
      }
    }
  }
}
</script>

<style lang="scss">
.quill-editor{
  background: #fff;
  .ql-editor{
    min-height: 300px;
  }
}
</style>
