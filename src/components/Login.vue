<template>
    <div class="login">
      <!--
        el-form : 整个form组件
        :model="form"  : 必须提供的对象
        label-width="80px"  : label宽度
        el-form-item    :  一个表单项
        element-ui中所存的组件在渲染的时候, 这个组件名就是类名
       -->
      <el-form ref="form" :model="form" label-width="80px">
        <img src="@/assets/avatar.jpg" alt="">
        <el-form-item label="用户名">
          <el-input v-model="form.username"></el-input>
        </el-form-item>
          <el-form-item label="密码">
          <el-input v-model="form.password"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">登录</el-button>
          <el-button>重置</el-button>
        </el-form-item>
      </el-form>
    </div>
</template>

<script>
export default {
  data () {
    return {
      form: {
        username: '',
        password: ''
      }
    }
  },
  methods: {
    onSubmit () {
      console.log('submit!')
    }
  }
}
</script>

<style lang="less">
// 注意：vue-cli把所有的loader都配置好了，但是没有把依赖项全部都下载下来
//  如果用less  需要安装 less  less-loader
.login{
  height: 100%;
  width: 100%;
  background-color:  #2d434c;
  overflow: hidden;
  .el-form{
  width: 400px;
  background-color: #fff;
  margin: 200px auto;
  padding: 75px 40px 15px;
  position: relative;
  img{
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    top: -70px;
    border-radius: 50%;
    border: 8px solid #fff;
  }
}
}

</style>
