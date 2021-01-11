<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddRoles">添加角色</el-button>
        </el-col>
      </el-row>
     <el-table :data='rolesList'>
       <el-table-column type="expand">
         <template slot-scope="scope">
           <el-row v-for="(item1,i1) in scope.row.children" :key="item1.id" :class="['bottomBorder',i1==0 ? 'topBorder':'','center']">
             <el-col :span="5">
               <el-tag closable @close="delRights(scope.row,item1.id)">{{item1.authName}}</el-tag>
               <i class="el-icon-caret-right"></i>
             </el-col>
             <el-col :span="19">
               <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="[i2==0?'':'topBorder','center']">
                 <el-col :span="6">
                   <el-tag type="success" closable @close="delRights(scope.row,item2.id)">{{item2.authName}}</el-tag>
                   <i class="el-icon-caret-right"></i>
                 </el-col>
                 <el-col :span="18" >
                   <el-tag type="danger" v-for="(item3) in item2.children" :key="item3.id" closable @close="delRights(scope.row,item3.id)">{{item3.authName}}</el-tag>
                 </el-col>
               </el-row>
             </el-col>
           </el-row>
         </template>
       </el-table-column>
       <el-table-column type="index" width="50"></el-table-column>
        <el-table-column prop="roleName" label="角色名称" >
        </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述">
        </el-table-column >
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-search" size="mini" @click="showEditRilesDialog(scope.row)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="delRoles(scope.row)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightsDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
     </el-table>
    </el-card>

    <el-dialog title="分配权限-" :visible.sync="setRightsDialog" width="50%" @close="clearRights">
      <el-tree :data="rolesRightsTree" :props="treeProps" show-checkbox node-key='id' default-expand-all :default-checked-keys='checkedRights' ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightsDialog = false">取 消</el-button>
        <el-button type="primary" @click="allotRights" >确 定</el-button>
      </span>
    </el-dialog>

  <!-- 添加角色的对话框 -->
    <el-dialog title="添加角色" :visible.sync="addRolesDialog" width="50%" @close="clearRights">
      <el-form :model="addRolesForm" :rules="rules" ref="editRolesFormRef" label-width="100px" class="demo-ruleForm">
        <el-form-item label="角色名称" prop="roleName" >
          <el-input v-model="addRolesForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRolesDialog = false">取 消</el-button>
        <el-button type="primary" @click="addRoles" >确 定</el-button>
      </span>
    </el-dialog>

    <!-- 这是角色编辑页面的弹出对话框 -->
    <el-dialog title="修改角色" :visible.sync="editRolesDialog" width="50%">
      <span>
        <el-form :model="editRolesForm" :rules="rules" ref="editRolesFormRef" label-width="100px" class="demo-ruleForm">
          <el-form-item label="角色名称" prop="roleName" >
            <el-input v-model="editRolesForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editRolesForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRolesDialog = false">取 消</el-button>
        <el-button type="primary" @click="editRoles">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      setRightsDialog: false,
      rolesRightsTree: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      checkedRights: [],
      readyId: '',
      editRolesDialog: false,
      editRolesForm: {},
      rules: {
        roleName: [
          { required: true, message: '角色名称不允许为空', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '角色描述不允许为空', trigger: 'blur' }
        ]
      },
      addRolesDialog: false,
      addRolesForm: {
        roleName: '',
        roleDesc: ''
      }
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    getRolesList () {
      this.axios.get('roles').then(res => {
        this.rolesList = res.data.data
      })
    },
    delRights (rol, id) {
      this.$confirm('此操作将取消相关权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.axios.delete(`roles/${rol.id}/rights/${id}`).then(res => {
          if (res.data.meta.status === 200) {
            this.$Message.success('删除权限成功')
            rol.children = res.data.data
          }
        })
      }).catch(() => {
        return true
      })
    },
    getCheckedBox (node, arry) {
      if (!node.children) {
        return arry.push(node.id)
      }
      node.children.forEach(element => {
        this.getCheckedBox(element, arry)
      })
    },
    showSetRightsDialog (role) {
      this.readyId = role.id
      this.getCheckedBox(role, this.checkedRights)
      this.setRightsDialog = true
      this.axios.get('rights/tree').then(res => {
        this.rolesRightsTree = res.data.data
      })
    },
    clearRights () {
      this.checkedRights = []
    },
    allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      this.axios.post(`roles/${this.readyId}/rights`, { rids: idStr }).then(res => {
        if (res.data.meta.status === 200) {
          this.setRightsDialog = false
          this.getRolesList()
          return this.$Message.success('更新成功')
        } else {
          return this.$Message.error('更新失败')
        }
      })
    },
    showEditRilesDialog (role) {
      this.editRolesDialog = true
      this.axios.get(`roles/${role.id}`).then(res => {
        this.editRolesForm = res.data.data
      })
    },
    editRoles () {
      this.$refs.editRolesFormRef.validate(valid => {
        if (!valid) {
          return true
        } else {
          this.axios.put('roles/' + this.editRolesForm.roleId, {
            roleName: this.editRolesForm.roleName,
            roleDesc: this.editRolesForm.roleDesc
          }).then(res => {
            if (res.data.meta.status === 200) {
              this.editRolesDialog = false
              this.$Message.success('更新成功')
              this.getRolesList()
            } else {
              this.$Message.error('更新失败')
            }
          })
        }
      })
    },
    showAddRoles () {
      this.addRolesDialog = true
    },
    addRoles () {
      this.axios.post('roles', this.addRolesForm).then(res => {
        if (res.data.meta.status === 201) {
          this.addRolesDialog = false
          this.getRolesList()
          this.$Message.success(res.data.meta.msg)
        } else {
          this.$Message.error('添加角色失败')
        }
      })
    },
    delRoles (role) {
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.axios.delete('roles/' + role.id).then(res => {
          if (res.data.meta.status === 200) {
            this.getRolesList()
            this.$Message.success(res.data.meta.msg)
          } else {
            this.$Message.error('删除失败')
          }
        })
      }).catch(() => {
        return true
      })
    }
  }
}
</script>
<style lang="less" scoped>
.el-tag{
  margin: 7px;
}

.topBorder{
  border-top: 1px solid #eee;
}

.bottomBorder{
  border-bottom:1px solid #eee
}

.center{
  display: flex;
  align-items: center;
}
</style>
