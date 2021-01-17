<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card @keyup.enter="searchGoods">
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            class="input-with-select"
            clearable @clear="getGoodsList"
          >
          <el-button slot="append" icon="el-icon-search" @click="searchGoods" ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addGoods">添加商品</el-button>
        </el-col>
      </el-row>
      <el-table border stripe :data="goodsList">
        <el-table-column type="index" width="50"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称" width="450">
        </el-table-column>
        <el-table-column prop="goods_price" label="商品价格（元）" width="130">
        </el-table-column>
        <el-table-column prop="goods_number" label="商品数量" width="100">
        </el-table-column>
        <el-table-column prop="goods_weight" label="商品重量（kg）" width="140">
        </el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="200">
          <template slot-scope="scope">
            {{scope.row.add_time|dateFormat}}
          </template>
        </el-table-column>
        <el-table-column  label="操作">
          <template slot-scope="scope">
              <el-button size="mini" type="primary" icon="el-icon-edit"></el-button>
              <el-button size="mini" type="danger" icon="el-icon-delete" @click="delGoods(scope.row)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 20, 30]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total" background>
    </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      goodsList: [],
      total: 0
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    getGoodsList () {
      this.axios.get('goods', { params: this.queryInfo }).then(res => {
        this.goodsList = res.data.data.goods
        this.total = res.data.data.total
      })
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getGoodsList()
    },
    searchGoods () {
      this.getGoodsList()
    },
    delGoods (goods) {
      console.log(goods)
      this.$confirm('此操作将永久删除该商品, 是否继续?\', \'提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.axios.delete(`goods/${goods.goods_id}`).then(res => {
          if (res.data.meta.status === 200) {
            this.$Message.success('删除成功')
            this.total--
            if (this.total % this.queryInfo.pagesize === 0) {
              this.queryInfo.pagenum--
            }
            this.getGoodsList()
          } else {
            this.$Message.error('删除失败')
          }
        })
      }).catch(() => {
        return true
      }
      )
    },
    addGoods () {
      this.$router.push('/goods/add')
    }
  }
}
</script>
