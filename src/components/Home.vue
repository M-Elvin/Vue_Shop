<template>
  <el-container class="home-container">

    <!-- 头部组件 -->
    <el-header>
      <div>
        <div class="head-logo"></div>
        <div>电商后台管理系统</div>
      </div>
      <el-button type="info" @click="logout">注销</el-button>
    </el-header>
    <el-container>

      <!-- 侧边栏 -->
      <el-aside :width="isCollpase? '60px' : '180px'">
        <div class="toggle-button" @click="toggle">|||</div>
        <el-menu
          :default-active="this.$route.path"
          class="el-menu-vertical-demo"
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409FFF"
          unique-opened
          :collapse="isCollpase"
          :collapse-transition="false"
          router 
          >
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id">
                  <template slot="title">
              <i class="el-icon-menu"></i>
              <span>{{subItem.authName}}</span>
            </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>

      <!-- 内容栏 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data(){
    return {
      // 左侧菜单数据
      menuList:[],
      iconsObj:{
        '125':'iconfont icon-user',
        '103':'iconfont icon-tijikongjian',
        '101':'iconfont icon-shangpin',
        '102':'iconfont icon-danju',
        '145':'iconfont icon-baobiao'
      },
      isCollpase:false
    }
  },
  created(){
    this.getMenuList()
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$message.info('成功注销')
      this.$router.push('/login')
    },
    async getMenuList(){
        const {data:res}=await this.$http.get('menus')
        if(res.meta.status !==200) return this.$message.error(res.meta.msg)
        this.menuList=res.data
        // console.log(res);
    },
    toggle(){
        this.isCollpase=!this.isCollpase
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #373d41;
  padding-left: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  > div {
    display: flex;
    align-items: center;
    color: #fff;
  }
  .head-logo {
    height: 60px;
    width: 160px;
    background-color: #fff;
    background: url('../assets/logo2.jpg') no-repeat center;
    background-size: cover;
  }
}
.el-aside {
  background-color: #333744;
  padding: 0;
  .el-menu{
    border-right: none;
  }
}
.el-main {
  background-color: #eaedf1;
}
.iconfont{
  margin-right: 10px;
}
.toggle-button{
  background-color: #4A5064;
  font-size: 10px;
  text-align: center;
  line-height: 24px;
  color: #fff;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
