<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input placeholder="请输入内容" v-model="params.query" clearable @clear="initUser">
            <el-button slot="append" icon="el-icon-search" @click="searchUser" class="search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
          <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
            <span>
              <el-form :model="addUserForm" :rules="rules" ref="addUserFormRef" label-width="100px" class="demo-ruleForm">
                <el-form-item label="用户名" prop="username">
                  <el-input v-model="addUserForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                  <el-input v-model="addUserForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                  <el-input v-model="addUserForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机号" prop="mobile">
                  <el-input v-model="addUserForm.mobile"></el-input>
                </el-form-item>
              </el-form>
            </span>
            <span slot="footer" class="dialog-footer">
              <el-button @click="addDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>
        </el-col>
      </el-row>
      <el-table :data="UserInfo" border stripe>
        <el-table-column type="index" width="50"></el-table-column>
        <el-table-column prop="username" label="姓名" >
        </el-table-column>
        <el-table-column prop="email" label="邮箱" >
        </el-table-column>
        <el-table-column prop="mobile" label="电话" >
        </el-table-column>
        <el-table-column prop="role_name" label="角色">
        </el-table-column>
        <el-table-column  label="状态">
          <template slot-scope="scope">
              <el-switch v-model="scope.row.mg_state" @change="stateChanged(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column  label="操作">
          <template slot-scope="scope">
            <el-tooltip class="item" effect="dark" content="修改信息" placement="top" :enterable="false">
              <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditForm(scope.row.id)"></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="删除用户" placement="top" :enterable="false">
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUser(scope.row.id)"></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="分配权限" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini" @click="showAssignRightsDialog(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="params.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size="params.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="UserNum">
    </el-pagination>
    </el-card>

    <el-dialog title="修改用户信息" :visible.sync="editDialogVisible" width="50%" @close="clearEditDialog">
     <span>
       <el-form :model="editUserForm" :rules="rules" ref="editUserFormRef" label-width="100px" class="demo-ruleForm">
        <el-form-item label="用户名" prop="username" >
          <el-input v-model="editUserForm.username" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editUserForm.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="editUserForm.mobile"></el-input>
        </el-form-item>
      </el-form>
     </span>
     <span slot="footer" class="dialog-footer">
       <el-button @click="editDialogVisible = false">取 消</el-button>
       <el-button type="primary" @click="editUser">确 定</el-button>
     </span>
    </el-dialog>

    <el-dialog title="分配权限" :visible.sync="assignRightsDialog" width="50%">
      <div>
        <p>当前用户：{{userInfo.username}}</p>
        <p>当前角色：{{userInfo.role_name}}</p>
        <p>分配新角色：
          <el-select v-model="selectedRoleId" placeholder="请选择">
            <el-option v-for="item in roleList" :key="item.id" :label="item.roleName" :value="item.id">
            </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="assignRightsDialog = false">取 消</el-button>
        <el-button type="primary" @click="saveNewRole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    var emailCheck = (rules, value, callback) => {
      const emailReg = /^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
      if (emailReg.test(value)) {
        return callback()
      } else {
        callback(new Error('请输入合法邮箱'))
      }
    }
    var mobileCheck = (rules, value, callback) => {
      const mobileReg = /^[1](([3][0-9])|([4][5-9])|([5][0-3,5-9])|([6][5,6])|([7][0-8])|([8][0-9])|([9][1,8,9]))[0-9]{8}$/
      if (mobileReg.test(value)) {
        return callback()
      } else {
        callback(new Error('请输入合法手机号'))
      }
    }

    return {
      UserInfo: [],
      UserNum: 1,
      params: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      addDialogVisible: false,
      addUserForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editUserForm: {
      },
      editDialogVisible: false,
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入活动名称', trigger: 'blur' },
          { min: 6, max: 20, message: '密码长度在 6 到 20 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: emailCheck, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: mobileCheck, trigger: 'blur' }
        ]
      },
      assignRightsDialog: false,
      userInfo: {},
      roleList: [],
      selectedRoleId: ''
    }
  },
  created () {
    this.initUser()
  },
  methods: {
    initUser () {
      this.axios.get('users', { params: this.params }).then(res => {
        this.UserInfo = res.data.data.users
        this.UserNum = res.data.data.total
      })
    },
    handleSizeChange (newSize) {
      this.params.pagesize = newSize
      this.initUser()
    },
    handleCurrentChange (newPage) {
      this.params.pagenum = newPage
      this.initUser()
    },
    stateChanged (row) {
      this.axios.put(`users/${row.id}/state/${row.mg_state}`).then(res => {
        if (res.status !== 200) {
          row.mg_state = !row.mg_state
          this.$Message.error('修改状态失败')
        } else {
          this.$Message.success('修改状态成功')
        }
      })
    },
    searchUser () {
      this.initUser()
    },
    addDialogClosed () {
      this.$refs.editUserFormRef.resetFields()
    },
    addUser () {
      this.$refs.addUserFormRef.validate(valid => {
        if (!valid) {
          return true
        } else {
          this.axios.post('users', this.addUserForm).then(res => {
            if (res.data.meta.status === 201) {
              this.$Message.success('添加用户成功')
              this.initUser()
              this.addDialogVisible = false
            } else {
              this.$Message.error('添加用户失败')
            }
          })
        }
      })
    },
    showEditForm (id) {
      this.editDialogVisible = true
      this.axios.get(`users/${id}`).then(res => {
        this.editUserForm = res.data.data
      })
    },
    editUser () {
      this.$refs.editUserFormRef.validate(valid => {
        if (!valid) {
          return true
        } else {
          this.axios.put('users/' + this.editUserForm.id, {
            email: this.editUserForm.email,
            mobile: this.editUserForm.mobile
          }).then(res => {
            if (res.data.meta.status === 200) {
              this.editDialogVisible = false
              this.$Message.success('更新成功')
              this.initUser()
            } else {
              this.$Message.error('更新失败')
            }
          })
        }
      })
    },
    clearEditDialog () {
      this.$refs.editUserFormRef.resetFields()
    },
    removeUser (id) {
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.axios.delete(`users/${id}`).then(res => {
          if (res.data.meta.status === 200) {
            this.$Message.success('删除成功')
          }
        })
      }).catch(() => {
        return true
      })
    },
    showAssignRightsDialog (userInfo) {
      this.userInfo = userInfo
      this.axios.get('roles').then(res => {
        this.roleList = res.data.data
      })
      this.assignRightsDialog = true
    },
    saveNewRole () {
      if (!this.selectedRoleId) {
        return this.$Message.error('请选择要分配的角色')
      } else {
        this.axios.put(`users/${this.userInfo.id}/role`, {
          rid: this.selectedRoleId
        }).then(res => {
          if (res.data.meta.status === 200) {
            this.assignRightsDialog = false
            this.initUser()
            return this.$Message.success('分配角色成功！')
          } else {
            return this.$Message.error(res.data.meta.msg)
          }
        })
      }
    }
  }
}
</script>

<style lang="less" scoped>
.search{
  background-color: #66B1FF !important;
  color: #fff !important;
}
</style>
