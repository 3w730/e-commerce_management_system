<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false" show-icon>
        </el-alert>
      <el-row class="distance">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader expand-trigger="hover" :options="cateList" v-model="selectedCateKeys" @change="handleCateChange" :props="cateProps" clearable></el-cascader>
        </el-col>
      </el-row>
      <el-tabs type="border-card" v-model="activeName" @tab-click="handleParamsClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini"  :disabled="isDisabled" @click="showAddDialog">添加参数</el-button>
          <el-table :data="manyTableList" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,index) in scope.row.attr_vals" :key="index" class="tag" closable @close="delTag(index,scope.row)">{{item}}</el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)" style="width:100px">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)" type="primary">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" width="50"></el-table-column>
            <el-table-column  prop="attr_name" label="参数名称" >
            </el-table-column>
            <el-table-column  label="操作" >
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">修改</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="delParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isDisabled" @click="showAddDialog">添加属性</el-button>
          <el-table :data="onlyTableList" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,index) in scope.row.attr_vals" :key="index" class="tag" closable @close="delTag(index,scope.row)">{{item}}</el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)" style="width:100px">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)" type="primary">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" width="50"></el-table-column>
            <el-table-column  prop="attr_name" label="属性名称" >
            </el-table-column>
            <el-table-column  label="操作" >
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">修改</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="delParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <el-dialog :title='"添加"+titleText' :visible.sync="addDialogVisible" width="50%" @close="clearAddDialog">
      <el-form ref="addParamsFormRef" :model="addParamsForm" label-width="100px" :rules="rules">
        <el-form-item :label='titleText' prop="attr_name">
          <el-input v-model="addParamsForm.attr_name" ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

<!-- 编辑部份 -->
    <el-dialog :title='"修改"+titleText' :visible.sync="editDialogVisible" width="50%" @close="clearEditDialog">
      <el-form ref="editParamsFormRef" :model="editParamsForm" label-width="100px" :rules="rules">
        <el-form-item :label='titleText' prop="attr_name">
          <el-input v-model="editParamsForm.attr_name" ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      cateList: [],
      selectedCateKeys: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      activeName: 'many',
      cateId: 0,
      manyTableList: [],
      onlyTableList: [],
      titleText: '动态参数',
      addDialogVisible: false,
      addParamsForm: {
        attr_name: ''
      },
      editDialogVisible: false,
      editParamsForm: {
        attr_name: ''
      },
      rules: {
        attr_name: [
          { required: true, message: '抱歉，不允许为空', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    getCateList () {
      this.axios.get('categories', { params: { type: 3 } }).then(res => {
        this.cateList = res.data.data
      })
    },
    handleCateChange () {
      if (this.selectedCateKeys.length < 3) {
        this.selectedCateKeys = []
        this.manyTableList = []
        this.onlyTableList = []
      } else {
        this.cateId = this.selectedCateKeys[2]
        this.getParamsList()
      }
    },
    handleParamsClick () {
      this.getParamsList()
    },
    getParamsList () {
      this.axios.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } }).then(res => {
        res.data.data.forEach(item => {
          item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
          item.inputVisible = false
          item.inputValue = ''
        })
        console.log(res)
        if (this.activeName === 'many') {
          this.manyTableList = res.data.data
          this.titleText = '动态参数'
        } else {
          this.onlyTableList = res.data.data
          this.titleText = '静态属性'
        }
      })
    },
    showAddDialog () {
      this.addDialogVisible = true
    },
    clearAddDialog () {
      this.$refs.addParamsFormRef.resetFields()
    },
    addParams () {
      this.$refs.addParamsFormRef.validate(valid => {
        if (!valid) {
          return true
        } else {
          this.axios.post(`categories/${this.cateId}/attributes`, { attr_name: this.addParamsForm.attr_name, attr_sel: this.activeName }).then(res => {
            if (res.data.meta.status === 201) {
              this.getParamsList()
              this.addDialogVisible = false
              this.$Message.success('添加成功')
            } else {
              this.$Message.error('添加失败')
            }
          })
        }
      })
    },
    clearEditDialog () {
      this.$refs.editParamsFormRef.resetFields()
    },
    showEditDialog (id) {
      this.axios.get(`categories/${this.cateId}/attributes/${id}`, { params: { attr_sel: this.activeName } }).then(res => {
        this.editParamsForm = res.data.data
      })
      this.editDialogVisible = true
    },
    editParams () {
      this.$refs.editParamsFormRef.validate(valid => {
        if (!valid) {
          return true
        } else {
          this.axios.put(`categories/${this.cateId}/attributes/${this.editParamsForm.attr_id}`, { attr_name: this.editParamsForm.attr_name, attr_sel: this.activeName }).then(res => {
            if (res.data.meta.status === 200) {
              this.editDialogVisible = false
              this.getParamsList()
              this.$Message.success('更新成功')
            } else {
              this.$Message.error('更新失败')
            }
          })
        }
      })
    },
    delParams (id) {
      this.$confirm(`此操作将永久删除该${this.titleText}, 是否继续?', '提示`, {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.axios.delete(`categories/${this.cateId}/attributes/${id}`).then(res => {
          console.log(res)
          if (res.data.meta.status === 200) {
            this.getParamsList()
            this.$Message.success('删除成功')
          } else {
            this.$Message.error('删除失败')
          }
        })
      }).catch(() => {
        return true
      })
    },
    handleInputConfirm (params) {
      if (params.inputValue.trim().length === 0) {
        params.inputVisible = false
        return true
      } else {
        params.attr_vals.push(params.inputValue)
        params.inputValue = ''
        params.inputVisible = false
        this.axios.put(`categories/${this.cateId}/attributes/${params.attr_id}`, {
          attr_name: params.attr_name,
          attr_sel: this.activeName,
          attr_vals: params.attr_vals.join(' ')
        }).then(res => {
          if (res.data.meta.status === 200) {
            this.$Message.success('添加成功')
          } else {
            this.$Message.error('添加失败')
          }
        })
      }
    },
    showInput (params) {
      params.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    delTag (index, params) {
      this.$confirm('此操作将永久删除该样式, 是否继续?\', \'提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(res => {
        params.attr_vals.splice(index, 1)
        this.axios.put(`categories/${this.cateId}/attributes/${params.attr_id}`, {
          attr_name: params.attr_name,
          attr_sel: this.activeName,
          attr_vals: params.attr_vals.join(' ')
        }).then(res => {
          if (res.data.meta.status === 200) {
            this.$Message.success('删除成功')
          } else {
            this.$Message.error('删除失败')
          }
        })
      })
    }
  },
  computed: {
    isDisabled () {
      if (this.selectedCateKeys.length !== 3) {
        return true
      } else {
        return false
      }
    }
  }
}
</script>
<style lang="less" scoped>
.distance{
  margin: 15px 0;
}

.tag{
  margin: 5px 8px;
}
</style>
