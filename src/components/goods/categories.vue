<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <tree-table :data="cateList" :columns="columns" :selection-type="false" show-index index-text="#" :show-row-hover="false" :expand-type="false" class="treeTable">
        <template slot-scope="scope" slot="state">
          <i class="el-icon-success" style="color:yellowgreen" v-if="scope.row.cat_deleted === true"></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>

        <template slot-scope="scope" slot="level">
          <el-tag v-if="scope.row.cat_level === 0 " >一级</el-tag>
          <el-tag v-else-if="scope.row.cat_level === 1" type="success">二级</el-tag>
          <el-tag v-else type="warning">三级</el-tag>
        </template>

        <template slot="option" slot-scope="scope">
          <el-button type="primary" size="mini" @click="showEditCateDialog(scope.row)">编辑</el-button>
          <el-button type="danger" size="mini" @click="delCate(scope.row)">删除</el-button>
        </template>
      </tree-table>
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[3, 5, 10, 15]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
       </el-pagination>
    </el-card>

    <el-dialog title="添加分类" :visible.sync="addCateDialog" width="40%"  @close="clearCateDialog">
      <el-form :model="addCateForm" :rules="rules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name" >
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader expand-trigger="hover" :options="addCateList"
          :props="addCateListProps" v-model="selectedCateId" @change="handleChange" clearable change-on-select>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialog = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑功能 -->
    <el-dialog
  title="修改分类"
  :visible.sync="editCateDialog"
  width="50%">
  <el-form :model="editCateForm" :rules="rules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name" >
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editCateDialog = false">取 消</el-button>
    <el-button type="primary" @click="editCate">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      cateList: [],
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '状态',
          type: 'template',
          template: 'state'
        },
        {
          label: '排序',
          type: 'template',
          template: 'level'
        },
        {
          label: '排序',
          type: 'template',
          template: 'option'
        }
      ],
      addCateDialog: false,
      rules: {
        cat_name: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ]
      },
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateList: [],
      addCateListProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedCateId: [],
      editCateDialog: false,
      editCateForm: {
        cat_name: ''
      }
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    getCateList () {
      console.log(this.queryInfo.pagenum)
      this.axios.get('categories', { params: this.queryInfo }).then(res => {
        this.cateList = res.data.data.result
        this.total = res.data.data.total
      })
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newNum) {
      this.queryInfo.pagenum = newNum
      this.getCateList()
    },
    showAddCateDialog () {
      this.axios.get('categories', { params: { type: 2 } }).then(res => {
        this.addCateList = res.data.data
      })
      this.addCateDialog = true
    },
    handleChange () {
      if (this.selectedCateId.length > 0) {
        this.addCateForm.cat_pid = this.selectedCateId[this.selectedCateId.length - 1]
        this.addCateForm.cat_level = this.selectedCateId.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
      console.log(this.addCateForm)
    },
    clearCateDialog () {
      this.$refs.addCateFormRef.resetFields()
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_name = ''
      this.selectedCateId = []
    },
    addCate () {
      this.axios.post('categories', this.addCateForm).then(res => {
        if (res.data.meta.status === 201) {
          this.$Message.success(res.data.meta.msg)
          this.getCateList()
          this.addCateDialog = false
        } else {
          this.$Message.error('添加分类失败')
        }
      })
    },
    showEditCateDialog (Cate) {
      this.editCateForm = Cate
      console.log(Cate)
      this.editCateDialog = true
    },
    editCate () {
      this.axios.put(`categories/${this.editCateForm.cat_id}`, { cat_name: this.editCateForm.cat_name }).then(res => {
        if (res.data.meta.status === 200) {
          this.$Message.success(res.data.meta.msg)
          this.getCateList()
          this.editCateDialog = false
        } else {
          this.$Message.error(res.data.meta.msg)
        }
      })
    },
    delCate (Cate) {
      this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.axios.delete(`categories/${Cate.cat_id}`).then(res => {
          if (res.data.meta.status === 200) {
            this.$Message.success(res.data.meta.msg)
            this.total--
            if (this.total % this.queryInfo.pagesize === 0) {
              this.queryInfo.pagenum--
            }
            this.getCateList()
          } else {
            this.$Message.error('删除失败')
          }
        })
      })
    }
  }
}
</script>
<style lang="less" scoped>
.treeTable{
  margin-top: 15px;
}

.el-cascader{
  width:100%
}

</style>
