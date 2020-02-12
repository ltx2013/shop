<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div class="left">
        <img src="../assets/heima.png" alt />
        <span>电商系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <!-- 侧边栏区域 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <!-- 侧边菜单栏区域 -->
        <div class="toggle-button" @click="collapse">|||</div>
        <el-menu
          background-color="#313743"
          text-color="#fff"
          active-text-color="#3a98ff"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="activePath"
        >
          <el-submenu
            :index="item.id + ''"
            v-for="item in menulist"
            :key="item.id"
          >
            <template slot="title">
              <i :class="iconobj[item.id]"></i>
              <span>{{ item.authName }}</span>
            </template>
            <el-menu-item
              :index="'/' + subitem.path"
              v-for="subitem in item.children"
              :key="subitem.id"
              @click="saveNavSate('/' + subitem.path)"
            >
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{ subitem.authName }}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 主体区域 -->
      <el-main>
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
      iconobj: {
        '125': 'iconfont icon-user',
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
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    // 退出登录
    logout() {
      window.localStorage.clear()
      this.$router.push('/login')
    },
    // 获取左侧菜单列表
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) {
        return this.$message.error('获取菜单数据失败')
      }
      this.menulist = res.data
    },
    // 收起左侧菜单
    collapse() {
      this.isCollapse = !this.isCollapse
    },
    // 保持连接的激活状态
    saveNavSate(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>
<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #363d40;
  padding-left: 0;
  .left {
    display: flex;
    align-items: center;
    span {
      margin-left: 25px;
      color: #fff;
      font-size: 18px;
    }
  }
}
.el-aside {
  background: #313743;
  user-select: none;
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background: #e9edf1;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  background: #475163;
  font-size: 14px;
  line-height: 24px;
  color: #e9edf1;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
