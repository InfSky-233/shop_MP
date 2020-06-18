<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-table :data="rolesList" stripe style="width: 100%" border>
        <el-table-column type="index" label="🔰"></el-table-column>
        <el-table-column prop="authName" label="角色名称"></el-table-column>
        <el-table-column prop="path" label="角色描述"></el-table-column>
        <el-table-column prop="level" label="操作">
          <template>
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="small"
              @click="changeUser(scope.row)"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="small"
              @click="openDelDialog(scope.row.id)"
            ></el-button>
            <!-- 角色分配按钮 -->
            <el-tooltip effect="dark" content="角色分配" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="small"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 角色列表
      rolesList: []
    }
  },
  created() {
    this.getRightsMenus()
  },
  methods: {
    getRightsMenus() {
      this.$http.get('menus').then(res => {
        console.log(res.data.data)
      })
    }
  }
}
</script>

<style lang="less" scoped>
</style>
