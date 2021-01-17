<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert title="添加商品信息" type="info" center show-icon></el-alert>
      <el-steps :space="300" :active="activeIndex-0" finish-status="success" align-center >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <el-form :model="addForm" :rules="rules" ref="addFormRef" label-width="100px" label-position="top">
        <el-tabs tab-position="left" v-model="activeIndex" @tab-click="switchTab" :before-leave="beforeTabLeave">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" ></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" ></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" ></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="attr_name">
              <el-cascader expand-trigger="hover" :options="cateList" v-model="selectedCatekeys" @change="handleChange" :props="cateProps">
              </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item v-for="item in manyTableData" :key="item.attr_id" :label="item.attr_name">
              <el-checkbox-group v-model="attrVals">
                <el-checkbox :label="c" v-for="(c,i) in item.attr_vals" :key="i" border ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload class="upload-demo" action="http://127.0.0.1:8888/api/private/v1/upload" :on-preview="handlePreview" :on-remove="handleRemove" :file-list="fileList" list-type="picture" :on-success="handleSuccess" :headers="imgHeaders">
             <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button type="primary" style="margin-top:15px" @click="addGoods">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <el-dialog title="提示" :visible.sync="imgDialog" width="50%">
      <img :src="imgUrl" style="width:100%">
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    var number = (rules, value, callback) => {
      const mobileReg = /^[1-9]\d*$/
      if (mobileReg.test(value)) {
        return callback()
      } else {
        callback(new Error('抱歉，只能为正整数'))
      }
    }
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        attrs: [],
        goods_introduce: ''
      },
      rules: {
        goods_name: [
          { required: true, message: '抱歉，不允许为空', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '抱歉，不允许为空', trigger: 'blur' },
          { validator: number, trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '抱歉，不允许为空', trigger: 'blur' },
          { validator: number, trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '抱歉，不允许为空', trigger: 'blur' },
          { validator: number, trigger: 'blur' }
        ]
      },
      cateList: [],
      selectedCatekeys: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      cateId: 0,
      manyTableData: [],
      onlyTableData: [],
      fileList: [],
      imgHeaders: {
        Authorization: window.sessionStorage.getItem('token')
      },
      imgDialog: false,
      imgUrl: '',
      attrVals: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    getCateList () {
      this.axios.get('categories').then(res => {
        this.cateList = res.data.data
      })
    },
    handleChange () {
      if (this.selectedCatekeys.length < 3) {
        this.selectedCatekeys = []
        return true
      }
      this.addForm.goods_cat = this.selectedCatekeys
      this.cateId = this.selectedCatekeys[2]
    },
    switchTab () {
      if (this.activeIndex === '1') {
        this.axios.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } }).then(res => {
          res.data.data.forEach(item => {
            item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
            this.manyTableData = res.data.data
            console.log(this.manyTableData)
          })
        })
      } else if (this.activeIndex === '2') {
        this.axios.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } }).then(res => {
          this.onlyTableData = res.data.data
        })
      }
    },
    beforeTabLeave (activeName, oldActiveName) {
      if (this.addForm.goods_cat.length !== 3) {
        this.$Message.error('请先选择商品分类')
        return false
      }
    },
    handlePreview (file) {
      this.imgUrl = file.response.data.url
      this.imgDialog = true
    },
    handleRemove (file) {
      const filePic = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex(x =>
        x.pic === filePic
      )
      this.addForm.pics.splice(i, 1)
    },
    handleSuccess (file) {
      const picInfo = { pic: file.data.tmp_path }
      this.addForm.pics.push(picInfo)
    },
    addGoods () {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) {
          return this.$Message.error('请先将内容填写完整')
        } else {
          this.addForm.goods_cat = this.addForm.goods_cat.join(',')
          this.attrVals.forEach((item, index) => {
            var id
            this.manyTableData.map(v => {
              if (v.attr_vals.indexOf(item) !== -1) {
                id = v.attr_id
                return id
              }
            })
            const attrInfo = { attr_id: id, attr_value: item }
            this.addForm.attrs.push(attrInfo)
          })

          this.onlyTableData.forEach(item => {
            const attrInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
            this.addForm.attrs.push(attrInfo)
          })

          this.axios.post('goods', this.addForm).then(res => {
            if (res.data.meta.status === 201) {
              this.$Message.success('添加商品成功')
              this.$router.push('/goods')
            } else {
              this.$Message.error('添加商品失败')
              this.$refs.addFormRef.resetFields()
              this.activeIndex = '0'
            }
          })
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
.el-steps{
  margin: 20px 0;
}

.el-checkbox{
  margin: 0 10px 0 0 !important;
}
</style>
