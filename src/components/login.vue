<template>
  <div class="login_container">
    <div class="login_box">
      <div class="login_avatar">
        <img src="../assets/imgs/chan.png" alt="">
      </div>
      <el-form class="login_form" :model="LoginForm" :rules="rules" ref="Login_form_ref">
        <el-form-item prop='username'>
          <el-input prefix-icon="iconfont icon-user" v-model='LoginForm.username'></el-input>
        </el-form-item>
        <el-form-item prop='password'>
          <el-input prefix-icon="iconfont icon-3702mima" v-model='LoginForm.password' type="password" ></el-input>
        </el-form-item>
        <el-form-item class="login_btn">
          <el-button type="primary" @click="login_">登录</el-button>
          <el-button type="info" @click='reset_login_form'>重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      LoginForm: {
        username: 'admin',
        password: '123456'
      },
      rules: {
        username: [
          { required: true, message: '请输入登录名称', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入活动名称', trigger: 'blur' },
          { min: 6, max: 20, message: '密码长度在 6 到 20 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    reset_login_form () {
      this.$refs.Login_form_ref.resetFields()
    },
    login_ () {
      this.$refs.Login_form_ref.validate((valid) => {
        if (valid) {
          this.axios.post('login', this.LoginForm).then(res => {
            console.log(res)
            if (res.data.meta.status !== 200) return this.$Message.error('用户名或或密码错误，登陆失败')
            this.$Message.success('登陆成功')
            window.sessionStorage.setItem('token', res.data.data.token)
            this.$router.push('/home')
          })
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}

.login_box{
  position: absolute;
  width: 450px;
  height: 300px;
  top: 50%;
  left: 50%;
  background-color: #fff;
  transform: translate(-50%,-50%);
  border-radius: 2px;

  .login_avatar{
    width: 120px;
    height: 120px;
    border-radius: 50%;
    position: absolute;
    left: 50%;
    transform: translate(-50%,-50%);
    background-color: rgb(102, 99, 99);
    padding: 15px;

    img{
      width: 100%;
      border-radius: 50%;
    }
  }
  .login_btn{
    display: flex;
    justify-content: flex-end;
  }
  .login_form{
    position: absolute;
    width: 100%;
    padding: 0px 20px;
    bottom: 0;
    box-sizing: border-box;
  }
}
</style>
