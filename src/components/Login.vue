<template>
  <div class="login_container">
    <div class="login_box">
      <div class="avatar_box">
        <!-- <img src="../assets/psb.jpg" alt=""> -->
        <div class="avatar_container"></div>
      </div>

      <div class="form_box">
        <el-form
          ref="loginformref"
          :rules="loginformRules"
          :model="Form"
          status-icon
          label-width="100px"
          class="form_container"
        >
          <el-form-item prop="username">
            <el-input
              prefix-icon="iconfont icon-user"
              type="text"
              v-model="Form.username"
            ></el-input>
          </el-form-item>
          <el-form-item prop="password">
            <el-input
              prefix-icon="iconfont icon-3702mima"
              type="password"
              v-model="Form.password"
            ></el-input>
          </el-form-item>

          <el-form-item>
            <el-button type="primary" @click="login">登录</el-button>
            <el-button @click="resetForm">重置</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
import { log } from 'util'
export default {
  data() {
    return {
      Form: {
        username: 'admin',
        password: '123456'
      },
      loginformRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 6, message: '用户名长度在3到6个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, message: '密码长度需要大于6个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // login(){
    //     this.$refs.loginformref.validate( valid=>{
    //         // console.log(valid);
    //         if(!valid)return;
    //         const {data:res}= await this.$http.post("login",this.Form);
    //         if(res.meta.status !==200)return console.log('登录失败');
    //         console.log('登录成功');
    //     });
    // },
    resetForm() {
    this.$refs.loginformref.resetFields()
    },
    login() {
    this.$refs.loginformref.validate(valid=>{
        if (!valid) return
        this.$http.post('login', this.Form).then(res => {
        const { data } = res
        if (data.meta.status !== 200) return this.$message.error('登录失败')
        this.$message.success('登录成功')
          // console.log(data);
        const { token } = data.data
          // 1.将登录成功后的token,保存到客户端的sessionStorage中
          // 1.1 项目中除了登录之外的其他API接口,必须在登录之后才能访问
          // 1.2 token 只应在当前网站打开期间生效,所以将 token 保存在sessionStorage中
        window.sessionStorage.setItem('token', token)
        this.$router.push('/home')
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  .avatar_box {
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;
    .avatar_container {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
      background: url('../assets/QQ图片20191114210855.jpg') no-repeat center;
      background-size: cover;
    }
  }
  .form_box {
    padding-top: 100px;
    width: 80%;
    text-align: center;
    .form_container {
      height: 100%;
      width: 100%;
      margin: 0 auto;
      /deep/input {
        height: 30px;
      }
    }
  }
}
</style>
