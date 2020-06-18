<template>
  <el-container class="home-container">
    <!-- 侧边栏 -->
    <el-aside :width="isCollapse ? '64px' : '240px'">
      <div :class="isCollapse ? 'fold' : 'open'">
        <img src="../assets/ta_logo.png" alt />
        <span>态势感知平台</span>
      </div>
      <!-- 侧边栏菜单区 -->
      <el-menu
        background-color="#333333"
        text-color="#fff"
        active-text-color="#1890ff"
        unique-opened
        :collapse="isCollapse"
        :collapse-transition="false"
        router
        :default-active="activePath"
      >
        <!-- 一级菜单 -->
        <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
          <template slot="title">
            <!-- 图标 -->
            <i :class="iconsItem[item.id]"></i>
            <!-- 文本 -->
            <span>{{item.authName}}</span>
          </template>
          <!-- 二级菜单 -->
          <el-menu-item
            :index="'/' + subItem.path"
            v-for="subItem in item.children"
            :key="subItem.id"
            @click="saveNavState('/' + subItem.path)"
          >
            <template slot="title">
              <i class="el-icon-menu"></i>
              <span>{{subItem.authName}}</span>
            </template>
          </el-menu-item>
        </el-submenu>
      </el-menu>
    </el-aside>
    <!-- 主体区域 -->
    <el-container>
      <!-- 头部区域 -->
      <el-header>
        <div class="toggle-button" @click="toggleCollapse">
          <i :class="isCollapse ? 'el-icon-s-unfold' : 'el-icon-s-fold'"></i>
        </div>
        <el-button type="info" @click="logout">退出</el-button>
      </el-header>
      <!-- 右侧主体区域 -->
      <el-main>
        <!-- Welcome路由占位符 -->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      // 左侧菜单数据
      menulist: [],
      iconsItem: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      // 是否折叠
      isCollapse: false,
      // 被激活高亮的链接地址
      activePath: ''
    }
  },
  created() {
    this.getMenyList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    getMenyList() {
      this.$http
        .get('menus')
        .then(res => {
          this.menulist = res.data.data
        })
        .catch(err => {
          this.$message.error(err.data.meta.msg)
        })
    },
    // 菜单折叠与展开
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    // 保存链接的激活状态
    saveNavState(activePath) {
      this.activePath = activePath
      window.sessionStorage.setItem('activePath', activePath)
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: space-between;
  .toggle-button {
    > i {
      font-size: 22px;
      color: #333333;
      cursor: pointer;
    }
  }
}
.el-aside {
  background-color: #333333;
  .open {
    height: 60px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 18px;
    letter-spacing: 3px;
    font-weight: bold;
    color: #ffffff;
    background-color: #333333;
    > img {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      margin-right: 20px;
    }
  }
  .fold {
    position: relative;
    height: 60px;
    > img {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
    > span {
      display: none;
    }
  }
  .el-menu {
    border-right: none;
    padding: 20px 0;
  }
}
.el-main {
  background-color: #f0f2f5;
}
.iconfont {
  font-size: 20px;
  margin-right: 10px;
}
</style>
