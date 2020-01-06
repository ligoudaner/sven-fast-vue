<template>
  <div class="site-wrapper site-page--login">
    <div class="site-content__wrapper">
      <div class="site-content">
        <div class="brand-info">
          <h2 class="brand-info__text"></h2>
          <p class="brand-info__intro"></p>
        </div>
        <div class="login-main">
          <h3 class="login-title">管理员登录</h3>
          <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" status-icon>
            <el-form-item prop="userName">
              <el-input v-model="dataForm.userName" placeholder="帐号"></el-input>
            </el-form-item>
            <el-form-item prop="password">
              <el-input v-model="dataForm.password" type="password" placeholder="密码"></el-input>
            </el-form-item>
            <el-form-item prop="captcha">
              <el-row :gutter="20">
                <el-col :span="14">
                  <el-input v-model="dataForm.captcha" placeholder="验证码">
                  </el-input>
                </el-col>
                <el-col :span="10" class="login-captcha">
                  <img :src="captchaPath" @click="getCaptcha()" alt="">
                </el-col>
              </el-row>
            </el-form-item>
            <el-form-item>
              <el-button class="login-btn-submit" type="primary" @click="dataFormSubmit()">登录</el-button>
            </el-form-item>
            <el-form-item>
              <el-button class="qq-login-btn" circle @click="qqLogin()">
                <icon-svg name="qq"></icon-svg>
              </el-button>
            </el-form-item>
          </el-form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import { getUUID } from '@/utils'
  export default {
    data () {
      return {
        dataForm: {
          userName: '',
          password: '',
          uuid: '',
          captcha: ''
        },
        dataRule: {
          userName: [
            { required: true, message: '帐号不能为空', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '密码不能为空', trigger: 'blur' }
          ],
          captcha: [
            { required: true, message: '验证码不能为空', trigger: 'blur' }
          ]
        },
        captchaPath: '',
        w: ''
      }
    },
    created () {
      this.getCaptcha()
    },
    methods: {
      // 提交表单
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl('/sys/login'),
              method: 'post',
              data: this.$http.adornData({
                'username': this.dataForm.userName,
                'password': this.dataForm.password,
                'uuid': this.dataForm.uuid,
                'captcha': this.dataForm.captcha
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$cookie.set('token', data.token)
                this.$router.replace({ name: 'home' })
              } else {
                this.getCaptcha()
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      // 获取验证码
      getCaptcha () {
        this.dataForm.uuid = getUUID()
        this.captchaPath = this.$http.adornUrl(`/captcha.jpg?uuid=${this.dataForm.uuid}`)
      },
      qqLogin () {
        this.$http({
          url: this.$http.adornUrl('/oauth2/getQqAuthorizeUrl'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.webSocket(data.state).then(() => {
              this.w = window.open(
                data.qqAuthorizeUrl, 'newwindow', 'height=500, width=400, top=' + (window.screen.availHeight - 500) / 2 +
                ', left = ' + (window.screen.availWidth - 400) / 2 + ', toolbar=no, menubar=no,scrollbars=no, resizable=no,location=no, status=no'
              )
            }, error => {
              this.$message.error(error.message)
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      webSocket (state) {
        if (WebSocket) {
          // 实例化socket
          this.socket = new WebSocket('ws://127.0.0.1:8080/sven-fast/socketServer/' + state)
          // 监听socket连接
          this.socket.onopen = this.openSocket
          // 监听socket错误信息
          this.socket.onerror = this.socketError
          // 监听socket消息
          this.socket.onmessage = this.getSocketMessage
          // 监听socket关闭
          this.socket.onclose = this.socketClose
        } else {
          return new Promise(function (resolve, reject) {
            reject(new Error('你的浏览器不支持WebSocket'))
          })
        }
        return new Promise(function (resolve, reject) {
          resolve()
        })
      },
      openSocket (evt) {
        console.log('连接成功')
      },
      getSocketMessage (evt) {
        if (this.w) {
          this.w.close()
          // this.$cookie.set('token', data.token)
          // this.$router.replace({ name: 'home' })
        }
        console.log('关闭窗口')
      },
      socketError (evt) {
        console.log('错误信息')
      },
      socketClose () {
        console.log('websocket关闭')
      }
    }
  }
</script>

<style lang="scss">
  .site-wrapper.site-page--login {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background-color: rgba(38, 50, 56, .6);
    overflow: hidden;
    &:before {
      position: fixed;
      top: 0;
      left: 0;
      z-index: -1;
      width: 100%;
      height: 100%;
      content: "";
      background-image: url(~@/assets/img/login_bg.jpg);
      background-size: cover;
    }
    .site-content__wrapper {
      position: absolute;
      top: 0;
      right: 0;
      bottom: 0;
      left: 0;
      padding: 0;
      margin: 0;
      overflow-x: hidden;
      overflow-y: auto;
      background-color: transparent;
    }
    .site-content {
      min-height: 100%;
      padding: 30px 500px 30px 30px;
    }
    .brand-info {
      margin: 220px 100px 0 90px;
      color: #fff;
    }
    .brand-info__text {
      margin:  0 0 22px 0;
      font-size: 48px;
      font-weight: 400;
      /*text-transform : uppercase;*/
    }
    .brand-info__intro {
      margin: 10px 0;
      font-size: 16px;
      line-height: 1.58;
      opacity: .6;
    }
    .login-main {
      position: absolute;
      top: 0;
      right: 0;
      padding: 150px 60px 180px;
      width: 470px;
      min-height: 100%;
      background-color: #fff;
    }
    .login-title {
      font-size: 16px;
    }
    .login-captcha {
      overflow: hidden;
      > img {
        width: 100%;
        cursor: pointer;
      }
    }
    .login-btn-submit {
      width: 100%;
      margin-top: 38px;
    }
    .qq-login-btn{
      padding: 1px;
      line-height: 1px;
    }
    .icon-svg {
      width: 2.2em;
      height: 2.2em;
      fill: currentColor;
      overflow: hidden;
      cursor: pointer;
    }
  }
</style>
