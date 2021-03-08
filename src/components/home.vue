<template>
  <el-container class="home-container">
    <!--头部区域-->
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="">
        <span>电商系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!--页面主题-->
    <el-container>
      <!--侧边栏-->
      <el-aside :width="istoggle ? '64px' : '200px'">
        <div class="toggle_button" @click="toggle">|||</div>
        <el-menu background-color="#333743" text-color="#fff" active-text-color="#409EFF" 
        :collapse="istoggle" :collapse-transition="false" :router="true">
        <!-- 一级菜单 -->
        <!-- 把item.id换格式-->
        <el-submenu :index="item.id + '' " v-for="item in menulist" :key="item.id">
        <template slot="title">
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>
        <!-- 2级菜单 -->
        <el-menu-item :index="'/' +subitem.path + '' " v-for="subitem in item.children" :key="subitem.id">
          <i class="el-icon-s-grid"></i>
          <span>{{subitem.authName}}</span>
        </el-menu-item>
        </el-submenu>
        </el-menu>
      </el-aside>
      <!--右侧内容-->
      <el-main>
        <!--路由占位符-->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>



</template>

<script>
export default {
  data() {
    return{
    //左侧菜单
    menulist:[],
    iconsObj: {
      '125':'el-icon-s-custom',
      '103':'el-icon-s-management',
      '101':'el-icon-s-shop',
      '102':'el-icon-s-claim',
      '145':'el-icon-s-data'
    },
    istoggle: false
    }
  },
  created() {
    this.getleft()
  },
  methods: {
    // 实现退出功能
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    //获取所有的菜单
    async getleft (){
      const {data:res} = await this.$http.get('menus')
      if( res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      console.log(res)
    },
    //切换折叠与展开
    toggle (){
      this.istoggle = ! this.istoggle
    }
  }
}
</script>

<style lang="less" scope>
.el-header{
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  > div{
    display: flex;
    align-items: center;
    span{
      margin-left: 15px;
    }
  }
}

.el-aside{
  background-color: #333744;
  .el-menu {
    border-right: 0;
  }
}

.el-main{
  background-color: #eaedf1;
}

.home-container{
  height: 100%;
  }

.toggle_button{
  background-color: #4A5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
