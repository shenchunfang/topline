<template>
  <div class="login-container">
    <div class="login-box">
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules">
        <el-form-item>
          <img src="./../../assets/logo_index.png" alt />
        </el-form-item>
        <el-form-item prop="mobile">
          <el-input placeholder="请输入手机号" v-model="loginForm.mobile">
            <i slot="prefix" class="iconfont icon-icon-test"></i>
          </el-input>
        </el-form-item>
        <el-form-item prop="code">
          <el-input placeholder="请输入密码" v-model="loginForm.code">
            <i slot="prefix" class="iconfont icon-mobile"></i>
          </el-input>
        </el-form-item>
        <el-form-item prop="xieyi">
          <el-checkbox style="margin-right:10px" v-model="loginForm.xieyi"></el-checkbox>
          <span>我已阅读并同意用户协议和隐私条款</span>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" style="width:100%" @click="login()" :disabled="isActive" :loading="isActive">登录</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
import '@/assets/iconfont/iconfont.css'
import '@/assets/js/gt.js'
export default {
  data () {
    var xieyiTest = function (rule, value, callback) {
      value ? callback() : callback(new Error('请勾选'))
    }
    return {
      isActive: false,
      ctaobj: null,
      loginForm: {
        mobile: '13911111111',
        code: '246810',
        xieyi: true
      },
      loginFormRules: {
        mobile: [
          { required: true, message: '手机号必填' },
          { pattern: /^1[35789]\d{9}$/, message: '手机号格式不正确' }
        ],
        code: [{ required: true, message: '密码必填' }],
        xieyi: [{ validator: xieyiTest }]
      }
    }
  },
  methods: {
    login () {
      // 全部表单域校验
      this.$refs.loginFormRef.validate(valid => {
        if (valid) {
          if (this.ctaobj !== null) {
            return this.ctaobj.verify()
          }
          // 获取极验秘钥请求信息
          this.isActive = true
          let pro = this.$http.get(`/captchas/${this.loginForm.mobile}`)
          pro
            .then(res => {
              // 接收处理返回的极验请求秘钥信息
              let { data } = res.data
              // console.log(data)
              // 显示极验验证窗口
              window.initGeetest(
                {
                  // 省略必须的配置参数
                  gt: data.gt,
                  challenge: data.challenge,
                  offline: !data.success,
                  new_captcha: true,
                  product: 'bind'
                },
                captchaObj => {
                  captchaObj
                    .onReady(() => {
                      // 验证码ready之后才能调用verify方法显示验证码
                      this.isActive = false
                      this.ctaobj = captchaObj
                      captchaObj.verify()
                    })
                    .onSuccess(() => {
                      // your code
                      this.loginAct()
                    })
                    .onError(() => {
                      // your code
                    })
                }
              )
            })
            .catch(err => {
              return this.$message.error('获取极验校验信息错误' + err)
            })
        }
      })
    },
    loginAct () {
      // 账号真实性校验
      var pro = this.$http.post('/authorizations', this.loginForm)
      pro
        .then(res => {
          // console.log(res)
          if (res.data.message === 'OK') {
            // 把信息存储到本地
            window.sessionStorage.setItem(
              'userinfo',
              JSON.stringify(res.data.data)
            )
            this.$router.push('/home')
          }
        })
        .catch(err => {
          return this.$message({
            type: 'error',
            message: '用户名或密码错误：' + err,
            duration: 1000
          })
        })
    }
  }
}
</script>

<style lang="less" scoped>
.login-container {
  height: 100%;
  background-image: url("./../../assets/login_bg.jpg");
  background-size: cover;
  display: flex;
  align-items: center;
  justify-content: center;
  .login-box {
    width: 400px;
    height: 350px;
    background-color: white;
    text-align: center;
    display: flex;
    align-items: center;
    justify-content: center;
    img {
      margin-top: 20px;
    }
    .el-form {
      width: 75%;
    }
  }
}
</style>
