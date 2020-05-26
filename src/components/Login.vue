<template>
  <div class="login_con">
    <div class="login_box">
      <!-- 头像 -->
      <div class="avatar_box">
        <img src="../assets/topic.jpg" alt="">
      </div>
      <!-- 登录表单 -->
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="0px" class="login_form">
        <!-- 用户名 -->
       <el-form-item prop="username">
        <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user" ></el-input>
       </el-form-item>
       <!-- 密码 -->
       <el-form-item prop="password">
        <el-input v-model="loginForm.password" prefix-icon="iconfont icon-3702mima" type="password"></el-input>
       </el-form-item>
       <!-- 登录 -->
       <el-form-item class="btns">
         <el-button type="primary" @click="login">登录</el-button>
         <el-button type="info" @click="resetLoginForm" >重置</el-button>
       </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>

export default {
  data () {
    return {
      //  这是登录表单的数据绑定对象
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 表单的验证规则
      loginFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 8, message: '长度在 3 到 8 个字符', trigger: 'blur' }

        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 9, message: '密码6-9位', trigger: 'blur' }
        ]
      }

    }
  },
  methods: {
    // 重置登录表单
    resetLoginForm () {
      // console.log(this)
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      this.$refs.loginFormRef.validate(async valid => {
        // console.log(valid) // 验证的结果第一个参数
        if (!valid) return // 为turn发送登录请求 false不发送
        const { data: res } = await this.$http.post('login', this.loginForm)
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error('登录失败！')
        this.$message.success('登录成功！')
        // 1.将登录成功之后的token，保存到客户端的sessionStorage中
        // 2.项目中除了登录之外的其他API接口，必须在登录之后才能访问
        // 3.token只应在当前的网站打开期间生效，所以将token保存在sessionStorage中
        // console.log(res)
        window.sessionStorage.setItem('token', res.data.token)
        // 4.通过编程式导航跳转到后台主页，路由地址是/home
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_con{
  height: 100%;
  background-color: #2b4b6b
}
.login_box{
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius:3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
  .avatar_box{
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius:50%;
    padding: 10px;
    box-shadow: 0 0 10 #ddd;
    position:absolute;
    left: 50%;
    transform: translate(-50%,-50% );
    background-color: #fff;
    img{
      width: 100%;
      height: 100%;
      border-radius:50%;

    }
  }
}
.login_form{
  position:absolute;
  bottom: 0;
  width: 100%;
  padding: 0 10px;
  box-sizing: border-box;
}
.btns{
  display: flex;
  justify-content: flex-end;
}
</style>
