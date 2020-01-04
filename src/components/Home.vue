<template>
      <el-container class="home-container">
    <el-header>
      <div>
        <i class="el-icon-monitor"></i>
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
      </el-header>
      <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏区域 -->
      <el-aside :width="isCollapse? '64px':'200px'">
         <div class="toggle-button" @click="toggleCollapse">|||</div>
        <!-- 侧边栏菜单 -->
        <el-menu background-color="#333744" text-color="#fff" active-text-color="#409eff" unique-opened :collapse='isCollapse' :collapse-transition="false" router :default-active="activePath">
          <!-- 一级菜单 -->
         <el-submenu :index="item.id+ ''" v-for="item in menulist" :key="item.id">
         <template slot="title">
           <!-- 图标区 -->
          <i :class="iconsObj[item.id]"></i>
          <!-- 文本 -->
          <span>{{item.authName}}</span>
         </template>
         <!-- 一级下的二级菜单 -->
         <el-menu-item :index=" '/'+k.path" v-for="k in item.children" :key="k.id" @click="saveNavStatus('/'+k.path)">
        <template slot="title">
          <i class="el-icon-menu"></i>
          <span>{{k.authName}}</span>
         </template>
        </el-menu-item>
        </el-submenu>
    </el-menu>
      </el-aside>
      <el-main>
        <!-- 路由占位符 -->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      menulist: [],
      iconsObj: {
        '125': 'iconfont icon-users',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activePath: ''
    }
  },

  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activPath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      //   console.log(this)

      this.$router.push('/login')
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      // console.log(this.menulist)
    },
    // 点击按钮切换菜单折叠展开
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    // 点击菜单动态赋值高亮
    saveNavStatus(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
.el-header{
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  >div{
    display: flex;
    align-items: center;
    span{
      margin-left: 15px;
    }
    i{
      font-size: 35px;
    }
  }
}
.home-container{
  height: 100%;
}
.el-aside{
  background-color: #333744;
  .el-menu{
  border-right: none
}
}
.el-main{
  background-color:#eaedf1;
}
.iconfont{
  margin-right: 10px
}
.toggle-button{
  background-color: #4a5064;
  color: #fff;
  text-align: center;
  font-size: 10px;
  line-height: 24px;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
