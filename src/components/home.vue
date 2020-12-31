<template lang="">
  <el-container class="home_container">
    <el-header>
      <div>
        <img src="../assets/imgs/chan.png"/>
        <span>电商管理系统</span>
      </div>
      <el-button type="info" @click="quit">退出</el-button></el-header>
    <el-container>
      <el-aside :width="isToggle ? '64px' : '200px'">
        <div class="togglebtn" @click="toggleMenu" v-html="toggleico"></div>
        <el-menu background-color="#333744" text-color="#fff" active-text-color="#409EFF" unique-opened :collapse="isToggle" :collapse-transition="false" router :default-active="activePath">
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconList[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="'/'+child.path"  v-for="child in item.children" :key="child.id" @click="saveIndex('/'+child.path)">
              <template slot="title">
                <i class="el-icon-s-grid"></i>
                <span>{{child.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
export default {
  data () {
    return {
      menuList: [],
      iconList: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-3702mima',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      toggleico: '&lt;=',
      isToggle: false,
      activePath: ''
    }
  },
  created () {
    this.getMenu()
    this.activePath = window.sessionStorage.getItem('path')
  },
  methods: {
    quit () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    getMenu () {
      this.axios.get('menus').then(res => {
        this.menuList = res.data.data
      })
    },
    toggleMenu () {
      if (this.isToggle) {
        this.toggleico = '&lt;='
      } else {
        this.toggleico = '=&gt;'
      }
      this.isToggle = !this.isToggle
    },
    saveIndex (path) {
      window.sessionStorage.setItem('path', path)
      this.activePath = path
    }
  }
}
</script>
<style lang="less" scoped>
.home_container{
  height: 100%;
}
.el-header{
  background-color:#373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;

  >div{
    display: flex;
    align-items: center;
    color: #fff;
    font-size: 18px;

    img{
      width: 25%;
    }
  }
}

.el-aside{
  background-color: #333744;

  .togglebtn{
    background-color: #4A5064;
    color: #fff;
    width: 100%;
    height: 25px;
    text-align: center;
    font-size: 18px;
    cursor: pointer;
  }
  .iconfont{
    margin-right: 10px;
  }

  .el-menu{
    border-right: none;
  }
}
</style>
