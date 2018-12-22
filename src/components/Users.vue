<template>
  <div class="users">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <el-input placeholder="请输入内容" v-model="query" class="input-with-select">
      <el-button slot="append" @click="search" icon="el-icon-search"></el-button>
    </el-input>
    <el-button type="success" plain @click="showAddDialog">添加用户</el-button>
    <!-- 表格组件 -->
    <!--
      el-table: 表格组件
      :data:  指定表格最终需要渲染的数据(数组)
      el-table-column:  定义表格的每一行
        label="日期"  列的标题
        prop:  对应显示的数据的属性名
        width: 列的宽度   不支持百分比

        在el-table中, 如果想要自定义列模板
        在 el-table-column中使用template, 指定属性slot-scope="scope"
          <template slot-scope="scope">自己定义的内容</template>
    -->
    <el-table :data="userList" style="width: 100%">
      <el-table-column prop="username" label="姓名" width="180"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="180"></el-table-column>
      <el-table-column prop="mobile" label="电话"></el-table-column>
      <el-table-column prop="mg_state" label="用户状态">
        <template slot-scope="scope">
          <!-- 在自定义模板中如何访问到当前列的数据 -->
          <!-- {{scope.row.mg_state}} -->
          <el-switch
            v-model="scope.row.mg_state"
            @change="changeState(scope.row)"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            @click="showEditDialog(scope.row)"
            plain
            size="mini"
          ></el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            @click="delUser(scope.row.id)"
            plain
            size="mini"
          ></el-button>
          <el-button type="success" icon="el-icon-check" plain size="mini">分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页
      @size-change: 表示每页的条数发生了改变, 会触发handleSizeChange事件
      @current-page:  当前页发生改变
      :current-page: 指定当前页面
      :page-sizes="[2, 4, 6, 8, 10]" :  指定每页显示的条数的数组
      :page-size="2"  :每页的条数
      total: 指定总条数
      layout: 指定分页的控件
    -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[2, 4, 6, 8, 10]"
      :page-size="2"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      background
    ></el-pagination>

    <!-- 添加用户的对话框
      el-dialog: 整个对话框
      visible:  对话框是否可见
    -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="40%">
      <!-- 添加表单 -->
      <el-form ref="addForm" :model="addForm" label-width="80px" :rules="rules" status-icon>
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" placeholder="请输入密码"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="addForm.mobile" placeholder="请输入手机号"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户信息的对话框
      el-dialog: 整个对话框
      visible:  对话框是否可见
    -->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="40%">
      <!-- 添加表单 -->
      <el-form ref="editForm" :model="editForm" label-width="80px" :rules="rules" status-icon>
        <el-form-item label="用户名">
          <el-tag type="info">{{editForm.username}}</el-tag>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="editForm.mobile" placeholder="请输入手机号"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
// 引入axios
// import axios from 'axios'
export default {
  data() {
    return {
      // 用户列表
      userList: [],
      query: '',
      currentPage: 1,
      pageSize: 2,
      total: 0,
      // 控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 表单校验规则
      rules: {
        // 用户名的校验规则
        username: [
          // 非空校验 trigger： blur  change
          { required: true, message: '用户名不能为空', trigger: 'blur' },
          {
            min: 3,
            max: 9,
            message: '用户长度在 3 到 9 个字符',
            trigger: 'blur'
          }
        ],
        password: [
          // 非空校验
          { required: true, message: '密码不能为空', trigger: 'blur' },
          {
            min: 6,
            max: 12,
            message: '用户长度在 3 到 9 个字符',
            trigger: 'blur'
          }
        ],
        email: [
          { type: 'email', message: '请输入一个合法的邮箱', trigger: 'blur' }
        ],
        mobile: [
          // 手机号正则校验
          {
            pattern: /^1\d{10}$/,
            message: '请输入一个合法的手机号',
            trigger: 'blur'
          }
        ]
      },
      // 控制修改用户对话框的显示与隐藏
      editDialogVisible: false,
      editForm: {
        id: '',
        username: '',
        email: '',
        mobile: ''
      }
    }
  },
  methods: {
    // 封装axios请求列表数据
    getUserList() {
      // axios 如果是get/delete请求, 参数要么直接拼地址栏, 要么放到params中
      //  如果是post/put/patch请求, 参数放到data中

      // 除了login请求, 其他所有接口都必须携带token, 要求设置给请求头: Authorization

      this.axios({
        methods: 'get',
        url: 'users',
        params: {
          query: this.query,
          pagenum: this.currentPage,
          pagesize: this.pageSize
        }
        // headers: {
        //   Authorization: localStorage.getItem('token')
        // }
      }).then(res => {
        // console.log(res)
        let {
          meta: { status },
          data: { users, total }
        } = res
        if (status === 200) {
          this.userList = users
          // console.log(this.userList)
          this.total = total
        }
      })
    },
    //  根据每页条数渲染列表
    handleSizeChange(val) {
      // console.log(`每页 ${val} 条`)
      // 修改this.pageSize
      this.pageSize = val
      this.getUserList()
    },
    // 根据当前页数渲染列表
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`)
      // 把currentPage修改成val
      this.currentPage = val
      this.getUserList()
    },
    // 搜索框功能
    search() {
      // 搜索的时候把当前页改成第一页, 从第一页开始搜索
      this.currentPage = 1
      this.getUserList()
    },
    // 删除功能
    delUser(id) {
      // console.log(id)
      // 删除时弹出提示框
      this.$confirm('您确定要删除吗?', '温馨提示', {
        type: 'warn'
      })
        .then(res => {
          // 发送axios请求, 删除对应id数据
          this.axios({
            method: 'delete',
            url: `users/${id}`
            // headers: {
            //   Authorization: localStorage.getItem('token')
            // }
          }).then(res => {
            // console.log(res)
            let {
              meta: { status }
            } = res
            if (status === 200) {
              // 删除成功
              // 如果发现当前页只有一条数据了, 应该currenPage-1, 渲染上一页
              if (this.userList.length <= 1 && this.currentPage > 1) {
                this.currentPage--
              }
              // 重新渲染
              this.getUserList()
              // 提示消息
              this.$message.success('删除成功')
            }
          })
        })
        .catch(() => {
          this.$message.info('取消删除')
        })
    },
    // 更改状态
    changeState({ id, mg_state: mgState }) {
      // console.log(user)
      // 发送ajax请求更改状态
      this.axios({
        method: 'put',
        url: `users/${id}/state/${mgState}`
        // headers: {
        //   Authorization: localStorage.getItem('token')
        // }
      }).then(res => {
        // console.log(res)
        let {
          meta: { status }
        } = res
        if (status === 200) {
          this.$message.success('状态修改成功')
        } else {
          this.$message.error('状态修改失败')
        }
      })
    },
    // 显示添加对话框
    showAddDialog() {
      this.addDialogVisible = true
    },
    // 添加用户
    addUser() {
      // 1. 表单校验功能
      this.$refs.addForm.validate(valid => {
        //   valid 表示表单是否通过校验 , 布尔值
        // console.log(valid)
        if (!valid) return false
        // 2. 发送ajax请求
        this.axios({
          method: 'post',
          url: 'users',
          data: this.addForm
        }).then(res => {
          // console.log(res)
          let {
            meta: { status, msg }
          } = res
          if (status === 201) {
            // 如果渲染时要显示新添加用户的那一页被渲染的页数
            this.total++
            this.currentPage = Math.ceil(this.total / this.pageSize)
            // 3. 重新渲染
            this.getUserList()
            // 重置表单样式
            this.$refs.addForm.resetFields()
            // 隐藏模态框
            this.addDialogVisible = false
            // 显示提示信息
            this.$message.success('添加成功了')
          } else {
            this.$message.error(msg)
          }
        })
      })
    },
    // 显示修改用户的对话框
    showEditDialog(row) {
      this.editDialogVisible = true
      // console.log(row)
      this.editForm.id = row.id
      this.editForm.username = row.username
      this.editForm.email = row.email
      this.editForm.mobile = row.mobile
    },
    // 更新表格修改的数据
    updateUser() {
      // 表单校验
      this.$refs.editForm.validate(valid => {
        // console.log(valid)
        if (!valid) return false
        this.axios({
          method: 'put',
          url: `users/${this.editForm.id}`,
          data: this.editForm
        }).then(res => {
          // console.log(res)
          let {
            meta: { status }
          } = res
          if (status === 200) {
            // 重新渲染
            this.getUserList()
            // 重置表单样式
            this.$refs.editForm.resetFields()
            // 关闭模态框
            this.editDialogVisible = false
            // 提示消息
            this.$message.success('修改成功')
          } else {
            this.$message.error('服务器异常')
          }
        })
      })
    }
  },
  created() {
    this.getUserList()
  }
}
</script>

<style lang="less" scoped>
.el-breadcrumb {
  height: 50px;
  line-height: 50px;
}
.input-with-select {
  width: 300px;
  margin-bottom: 10px;
}
</style>
