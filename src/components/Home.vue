<template> 
<el-container class="home-container">
  <!-- 头部区域 -->
  <el-header>
    <div><h2>电商后台管理系统</h2></div>
      <el-button type="info" @click="logOut">退出</el-button></el-header>
  <!-- 页面主题区域 -->
  <el-container>
    <!-- 侧边栏 -->
    <el-aside :width="isCollapse ? '64px' :'200px'">
      <div class="toggle-button" @click="toggleCollapse">|||</div>
      <!-- 侧边栏菜单区域 -->
      <el-menu
      background-color="#333744"
      text-color="#fff"
      active-text-color="#409EFF"
      :unique-opened="true"
      :collapse="isCollapse"
      :collapse-transition="false"
      :router="true"
      :default-active="activePath"
      >
      <!-- 一级菜单 -->
      <el-submenu :index="item.id+''" v-for="item in menulist " :key="item.id">
        <!-- 一级菜单的模板 -->
        <template slot="title">
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>
        <!-- 二级菜单 -->
      <el-menu-item :index="'/'+subitem.path+''" v-for="subitem in item.children" 
      :key="subitem.id" @click="saveNavState('/'+subitem.path)">
          <template slot="title">
          <i class="el-icon-menu"></i>
          <span>{{subitem.authName}}</span>
        </template>
      </el-menu-item>
      </el-submenu>
     
    </el-menu>
    </el-aside>
    <!-- 右边主体区域 -->
    <el-main><router-view></router-view></el-main>
  </el-container>
</el-container>

</template>

<script>
export default {
  created(){
    this.getMenuList()
    this.activePath=window.sessionStorage.getItem('activePath')
  },
  methods:{
    //退出功能实现
    logOut(){
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    //获取列表数据
   async getMenuList(){
    const {data:res}=  await this.$http.get('menus')
    if(res.meta.status!==200) return this.$message.error(res.meta.msg)
    this.menulist=res.data
    },
    //点击按钮实现菜单的切换和展开
    toggleCollapse(){
      this.isCollapse=!this.isCollapse
    },
    //实现点击侧边栏高亮
    saveNavState(activePath){
      window.sessionStorage.setItem('activePath',activePath)
      this.activePath=activePath
    }
  },
  data(){
    return{
      //左侧菜单数据
      menulist:[],
      iconsObj:{
        '125':'iconfont icon-user',
        '103':'iconfont icon-tijikongjian',
        '101':'iconfont icon-shangpin',
        '102':'iconfont icon-danju',
        '145':'iconfont icon-baobiao'
      },
      //是否折叠
      isCollapse:false,
      //被激活的链接地址
      activePath:''
    }

  }
}
</script>

<style  lang="less" scoped>
.home-container{
  height: 100%;
}
.el-header{
  display: flex;
  background-color: #373D41;
  justify-content: space-between;
  align-items: center;
  color: #fff;
}
.el-aside {
  background-color: #333744;
  .el-menu{
    border-right: none;
  }
}
.el-main{
  background-color: #eaedf1;
}
.iconfont{
  margin-right: 10px;
}

.toggle-button{
  background-color: #4A5064;
  color: #fff;
  line-height: 24px;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
  font-size: 10px
  

}

</style>