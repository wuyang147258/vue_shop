<template>
  <div id="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="">
      </div>
      <!-- 登录表单区域 -->
      <el-form  ref="loginFormRef" :model="loginForm" label-width="0px" class="login_form" :rules="loginFormRules">
        <!-- 用户名 -->
         <el-form-item label="" prop="username">
               <el-input  prefix-icon="el-icon-user" v-model="loginForm.username"></el-input>
        </el-form-item>
        <!-- 密码 -->
           <el-form-item label="" prop="password">
               <el-input  prefix-icon="el-icon-more-outline
" v-model="loginForm.password"  type="password"></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btns">
            <el-button type="primary" @click="login">登录</el-button>
             <el-button type="info" @click="resetFormLogin">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data(){
    return{
      //登录表单的数据绑定对象
      loginForm:{
        username:'admin',
        password:'123456'
      },
      //这是表单的验证规则对象
      loginFormRules:{
        //验证用户名是否合法
        username:[
         { required: true, message: '请输入登录名称', trigger: 'blur'},
         { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        //验证密码是否合法
        password:[
           { required: true, message: '请输入登录密码', trigger: 'blur' },
           { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods:{
    //点击后重置表单
    resetFormLogin(){
      this.$refs.loginFormRef.resetFields()
    },
    login(){
      console.log(1);
      //实现表单的预验证
      this.$refs.loginFormRef.validate( async valid=>{
        if(!valid)return;
       const {data:res}=await this.$http.post('login',this.loginForm)
       if(res.meta.status!==200) return this.$message.error('登录失败')
       this.$message.success('登录成功')
       //将登录成功的token保存到客户端的sessionStorage中
       window.sessionStorage.setItem("token",res.data.token);
       //通过编程式导航到首页
       this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
#login_container{
  background-color: #2b4b6b;
  height: 100%;
}
.login_box{
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%,-50%);
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  .avatar_box{
    position: absolute;
    left: 50%;
    transform: translate(-50%,-50%);
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    img{
      width: 100%;
      height: 100%;
       border-radius: 50%;
       background-color: #eee;
       
    }
  }
}
.login_form{
  position:absolute;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
  bottom: 0 ;
}
.btns{
  display: flex;
  justify-content: flex-end;
}
</style>