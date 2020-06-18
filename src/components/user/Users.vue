<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="30">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addUserDialog = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userlist" border stripe>
        <el-table-column type="index" label="🔰"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="250px">
          <template slot-scope="scope">
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
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 10, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 添加用户对话框 -->
    <el-dialog title="添加用户" :visible.sync="addUserDialog" width="30%" @close="addDialogClosed">
      <!-- 内容主体 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部按钮 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addUserDialog = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户信息"
      :visible.sync="changeUserDialog"
      width="30%"
      @close="changeDialogClosed"
    >
      <el-form
        :model="changeUserInfo"
        :rules="addFormRules"
        ref="changeFormRef"
        label-width="70px"
        status-icon
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="changeUserInfo.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="changeUserInfo.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="changeUserInfo.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="changeUserDialog = false">取 消</el-button>
        <el-button type="primary" @click="changeUserSure">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 删除用户对话框 -->
  </div>
</template>

<script>
export default {
  data() {
    // 验证邮箱规则
    var checkEmail = (rule, value, callback) => {
      // 验证邮箱正则
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        return callback()
      }
      callback(new Error('请输入正确的邮箱'))
    }
    // 验证手机号规则
    var checkMobile = (rule, value, callback) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) {
        return callback()
      }
      callback(new Error('请输入正确的手机号'))
    }
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userlist: [],
      total: 0,
      // 控制添加用户对话框的显示与隐藏
      addUserDialog: false,
      // 添加用户表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单的验证规则
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱地址', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 控制修改用户对话框的显示与隐藏
      changeUserDialog: false,
      // 当前需要修改用户的信息
      changeUserInfo: {
        username: '',
        email: '',
        mobile: '',
        id: ''
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    getUserList() {
      this.$http
        .get('users', { params: this.queryInfo })
        .then(res => {
          this.userlist = res.data.data.users
          this.total = res.data.data.total
        })
        .catch(() => {
          this.$message.error('获取用户数据失败！')
        })
    },
    // 监听 pagesize 改变的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听 页码值 改变的事件
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听 switch 状态的改变
    userStateChanged(userinfo) {
      this.$http
        .put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
        .then(res => {
          this.$message.success(res.data.meta.msg)
        })
        .catch(err => {
          this.$message.success(err.data.meta.msg)
        })
    },
    // 监听添加用户对话框关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 点击确定按钮，添加用户
    addUser() {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) return
        // 发起添加用户的网络请求
        this.$http
          .post('users', this.addForm)
          .then(() => {
            this.$message.success('添加用户成功！')
            // 隐藏添加用户对话框
            this.addUserDialog = false
            // 刷新用户列表数据
            this.getUserList()
          })
          .catch(() => {
            this.$message.error('添加用户失败！')
          })
      })
    },
    // 监听修改用户对话框关闭事件
    changeDialogClosed() {
      this.$refs.changeFormRef.resetFields()
    },
    // 点击编辑按钮，获取当前需要修改的用户信息
    changeUser(userinfo) {
      this.changeUserInfo.username = userinfo.username
      this.changeUserInfo.email = userinfo.email
      this.changeUserInfo.mobile = userinfo.mobile
      this.changeUserInfo.id = userinfo.id
      this.changeUserDialog = true
    },
    // 点击确定按钮，修改用户信息
    changeUserSure() {
      this.$refs.changeFormRef.validate(valid => {
        if (!valid) return
        // 发起修改用户信息的网络请求
        this.$http
          .put('users/' + this.changeUserInfo.id, {
            email: this.changeUserInfo.email,
            mobile: this.changeUserInfo.mobile
          })
          .then(res => {
            console.log(res)
            this.$message.success('修改用户信息成功！')
            // 隐藏修改用户信息对话框
            this.changeUserDialog = false
            // 刷新修改后用户列表数据
            this.getUserList()
            console.log(this.userlist)
          })
          .catch(() => {
            this.$message.error('修改用户信息失败！')
          })
      })
    },
    // 点击删除按钮， 根据id删除对应用户信息
    async openDelDialog(id) {
      const confirmResult = await this.$confirm(
        '此操作将删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult === 'confirm') {
        return this.$http
          .delete('users/' + id)
          .then(() => {
            this.$message.success('删除用户成功！')
            if (
              this.queryInfo.pagenum ===
              Math.ceil(this.total / this.queryInfo.pagesize)
            ) {
              if (this.total % this.queryInfo.pagesize === 1) {
                this.queryInfo.pagenum -= 1
              }
            }
            this.getUserList()
          })
          .catch(() => {
            this.$message.success('删除用户失败！')
          })
      }
      this.$message.info('已取消删除')
    }
  }
}
</script>

<style lang="less" scoped>
.el-table_1_column_7 > .cell .el-button {
  margin-right: 12px;
}
</style>
