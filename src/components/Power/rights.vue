<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-table :data='rightsList' border stripe>
        <el-table-column type="index" width="50"></el-table-column>
        <el-table-column prop="authName" label="权限名称" >
        </el-table-column>
        <el-table-column prop="path" label="路径">
        </el-table-column>
        <el-table-column prop="level" label="权限等级" >
          <template slot-scope="scope">
              <el-tag v-if="scope.row.level==0">一级</el-tag>
              <el-tag v-if="scope.row.level==1" type="success">二级</el-tag>
              <el-tag v-if="scope.row.level==2" type="warning">三级</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rightsList: []
    }
  },
  created () {
    this.getRightsList()
  },
  methods: {
    getRightsList () {
      this.axios.get('rights/list').then(res => {
        this.rightsList = res.data.data
      })
    }
  }
}
</script>

<style lang="less" scoped></style>
