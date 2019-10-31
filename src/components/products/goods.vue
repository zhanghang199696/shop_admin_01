<template>
  <div class="goods">
    <!-- 添加按钮 -->
    <el-button @click="goodsAdd" class="addbtn" type="success" plain>添加商品</el-button>
    <!-- 表格 -->
    <el-table :data="goodsList">
      <el-table-column :index="indexMethod" type="index"></el-table-column>
      <el-table-column label="商品名称" prop="goods_name"></el-table-column>
      <el-table-column label="商品价格" prop="goods_price"></el-table-column>
      <el-table-column label="商品重量" prop="goods_weight"></el-table-column>
      <el-table-column label="创建时间" prop="add_time">
        <template v-slot:default="{row}">
          {{ row.add_time | time }}
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template v-slot:default="{row}">
          <el-button type="primary" plain icon="el-icon-edit" size="small"></el-button>
          <el-button type="danger" plain icon="el-icon-delete" size="small"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页器 -->
     <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>
export default {
  data () {
    return {
      goodsList: [],
      pagenum: 1,
      pagesize: 5,
      total: 0
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList () {
      const { meta, data } = await this.$axios.get('goods', {
        params: {
          pagenum: this.pagenum,
          pagesize: this.pagesize
        }
      })
      if (meta.status === 200) {
        this.goodsList = data.goods
        this.total = data.total
      } else {
        this.$message.error(meta.msg)
      }
    },
    handleSizeChange (val) {
      this.pagesize = val
      this.pagenum = 1
      this.getGoodsList()
    },
    handleCurrentChange (val) {
      this.pagenum = val
      this.getGoodsList()
    },
    indexMethod (index) {
      return index + 1 + (this.pagenum - 1) * this.pagesize
    },
    goodsAdd () {
      this.$router.push('/goodsAdd')
    }
  }
}
</script>

<style lang="scss" scoped>
.goods{
  .addbtn{
    margin-bottom: 10px;
  }
}
</style>
