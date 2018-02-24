<template>
  <div class="login">
    <div class="login-panel clearfix">
      <div class="left">
        <img src="../../assets/image/logo_b.png" width="136" height="136">
        <p class="logo-desc">晶蝶云平台</p>
        <p class="phone-desc">免费服务热线：400-685-8188</p>
      </div>
      <div class="right">
        <div class="top-btn clearfix">
          <div class="btn-group">
            <button class="pwd-btn" :class="pwdActive?'active':''" @click="showPwd">账号密码登录</button>
            &nbsp;/&nbsp;
            <button class="code-btn" :class="codeActive?'active':''" @click="showCode">手机号验证登录</button>
          </div>
          <router-link to="/register" class="register-btn">注册</router-link>
        </div>
        <Form class="form" ref="formPwd" :model="formPwd" :rules="rulePwd" v-if="pwdActive">
          <Form-item prop="account">
            <Input type="text" class="phone" v-model="formPwd.account" placeholder="请输入手机号">
            </Input>
          </Form-item>
          <Form-item prop="password">
            <Input :type="lookPwd?'text':'password'" class="pwd" v-model="formPwd.password" placeholder="请输入密码">
            </Input>
            <i class="icon" @click="lookPwd = !lookPwd" :class="lookPwd?'':'active'"></i>
          </Form-item>
        </Form>
        <Form class="form" ref="formCode" :model="formCode" :rules="ruleCode" v-else>
          <Form-item prop="account">
            <Input type="text" class="phone" v-model="formCode.account" placeholder="请输入手机号">
            </Input>
            <button class="get-code" @click="getCode" v-if="!sendCode">获取验证码</button>
            <button class="get-code" v-else>{{timeOut}}S</button>
          </Form-item>
          <Form-item prop="code">
            <Input type="text" class="code" v-model="formCode.code" placeholder="请输入验证码">
            </Input>
          </Form-item>
        </Form>
        <div class="foot-btn clearfix">
          <Checkbox v-model="rememberMe" class="checkbox" @on-change="handleChange" v-if="pwdActive">记住我</Checkbox>
          <button class="submit" @click="handleSubmit('formPwd')" v-if="pwdActive">登录</button>
          <button class="submit" @click="handleSubmit('formCode')" v-else>登录</button>
          <router-link to="/forget" class="forget">忘记密码?</router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import * as api from '../../vuex/api'

export default {
  data() {
    return {
      formPwd: {
        account: '',
        password: ''
      },
      formCode: {
        account: '',
        code: ''
      },
      rulePwd: {
        account: [
          { required: true, message: '请填写手机号', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请填写密码', trigger: 'blur' },
          { type: 'string', min: 6, message: '密码长度不能小于6位', trigger: 'blur' }
        ]

      },
      ruleCode: {
        account: [
          { required: true, message: '请填写手机号', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请填写验证码', trigger: 'blur' },
          { type: 'string', min: 6, max: 6, message: '请输入6位验证码', trigger: 'blur' }
        ]
      },
      lookPwd: false,
      pwdActive: true,
      codeActive: false,
      rememberMe: false, // 记住我
      sendCode: false,
      timeOut: 60,
    }
  },
  created() {
    this.getFormInfo();
  },
  mounted() {
    let _this = this;
    document.onkeydown = function(e) {
      if (e.keyCode == 13) {
        if (_this.pwdActive) {
          _this.handleSubmit('formPwd');
        } else {
          _this.handleSubmit('formCode');
        }

      }
    }
  },
  methods: {
    getCode() {
      if (!this.formCode.account) {
        this.$Message.error('请填写正确的手机号');
        return false;
      } else {
        this.$http.post(api.LOGIN + 'sendSmsCodeByMerchant',
          JSON.stringify(this.getParam('code')))
          .then(res => {
            // console.log(res);
            let response = res.body;
            if (response.code === 0) {
              this.sendCode = true;
              this.setTimeOut();
              this.$Message.success('验证码已发送');
            }
          })
          .catch(err => {
            console.log(err);
          })
      }
    },
    getFormInfo() {
      if (localStorage.remember === 'true') {
        let formInfo = JSON.parse(localStorage.loginInfo);
        this.rememberMe = true;
        this.formPwd.account = formInfo.account;
        this.formPwd.password = formInfo.pwd;
      }
    },
    handleSubmit(name) {
      if (name === 'formPwd') {
        this.$refs[name].validate((valid) => {
          if (valid) {
            this.$http.post(api.LOGIN + 'login',
              JSON.stringify(this.getParam('login')))
              .then(res => {
                let response = res.body;
                if (response.code === 0) {
                  this.$store.dispatch('login', response.token);
                  this.$store.dispatch('userInfo', JSON.stringify(response.data));
                  this.$store.dispatch('account', this.formPwd.account);
//                  this.$store.dispatch('currentMerchantId', response.data.userData.merchantId);
                  this.$Message.success('登录成功');

                  if (this.rememberMe) {
                    localStorage.setItem('loginInfo', JSON.stringify({
                      account: this.formPwd.account,
                      pwd: this.formPwd.password
                    }));
                  }

                  setTimeout(() => {
                    this.$router.replace('/cloud');
                  }, 1000);
                } else if (response.code === 5003) {
                  this.$Message.error(response.showMsg)
                }else{
                  this.$Message.error(response.showMsg)
                }
              })
              .catch(err => {
                console.log(err);
              })
          } else {
            this.$Message.error('表单验证失败!');
          }
        })
      } else {
        this.$refs[name].validate((valid) => {
          if (valid) {
            this.$http.post(api.LOGIN + 'loginBySmsCode',
              JSON.stringify(this.getParam('loginBySmsCode')))
              .then(res => {
                // console.log(res)
                let response = res.body;
                if (response.code === 0) {
                  this.$store.dispatch('login', response.token);
                  this.$store.dispatch('userInfo', JSON.stringify(response.data));
                  this.$store.dispatch('account', this.formCode.account);
                  this.$store.dispatch('currentMerchantId', response.data.userData.merchantId);
                  this.$Message.success('登录成功');

                  setTimeout(() => {
                    this.$router.replace('/cloud');
                  }, 1000);
                } else if (response.code === 5003) {
                  this.$Message.error(response.showMsg)
                }
              })
              .catch(err => {
                console.log(err)
              })
          }
        })
      }
    },
    handleChange(value) {
      if (value) {
        localStorage.setItem('remember', 'true');

      } else {
        localStorage.removeItem('remember');
        localStorage.removeItem('account');
      }
    },
    showPwd() {
      this.pwdActive = true;
      this.codeActive = false;
    },
    showCode() {
      this.pwdActive = false;
      this.codeActive = true;
    },
    setTimeOut() {
      let timer = setTimeout(() => {
        this.setTimeOut();
        if (this.timeOut > 0) {
          this.timeOut--
        }
      }, 1000);
      if (this.timeOut <= 0) {
        this.sendCode = false;
        this.timeOut = 60;
        clearTimeout(timer)
      }
    },
    getParam(name) {
      if (name === 'code') {
        return {
          reqId: 1,
          param: {
            phone: this.formCode.account,
            type: 1,
            childType: 18
          }
        }
      } else if (name === 'login') {
        return {
          channel: 10,
          param: {
            account: this.formPwd.account,
            pwd: this.formPwd.password
          }
        }
      } else if (name === 'loginBySmsCode') {
        return {
          reqId: 1,
          channel: 10,
          param: {
            account: this.formCode.account,
            smsCode: this.formCode.code
          }
        }
      }
    }
  }
};
</script>

<style lang="scss" rel="stylesheet/scss">
.login {
  position: fixed;
  width: 100%;
  left: 0;
  top: 0;
  bottom: 0;
  z-index: 1;
  background: url(../../assets/image/login_bg.png) no-repeat;
  background-size: 100% 100%;
}

.login-panel {
  position: fixed;
  margin: -228px 0 0 -440px;
  left: 50%;
  top: 50%;
  z-index: 10;
  width: 880px;
  height: 456px;
  border-radius: 2px;
  background-color: #ffffff;
  .left {
    float: left;
    padding-top: 100px;
    width: 320px;
    height: 456px;
    background: url(../../assets/image/login_bg_red.png) no-repeat;
    img {
      display: block;
      margin: 0 auto 10px;
    }
    .logo-desc {
      margin-bottom: 100px;
      font-size: 24px;
      text-align: center;
      color: #fff;
    }
    .phone-desc {
      font-size: 20px;
      text-align: center;
      color: #fff;
    }
  }
  .right {
    float: right;
    padding: 24px 40px;
    width: 560px;
    height: 456px;
    .top-btn {
      margin-bottom: 160px;
      .btn-group {
        float: left;
        width: 350px;
        font-size: 16px;
        .pwd-btn,
        .code-btn {
          width: 160px;
          border: none;
          outline: none;
          height: 22px;
          line-height: 22px;
          text-align: center;
          background: transparent;
          color: #99a9bf;
          cursor: pointer;
          &.active {
            height: 28px;
            line-height: 28px;
            font-size: 20px;
            color: #1f2d3d;
          }
        }
      }
      .register-btn {
        float: right;
        display: block;
        width: 40px;
        text-align: right;
        color: #ff4949;
        font-size: 16px;
        cursor: pointer;
        text-decoration: none;
        &:hover {
          opacity: .9;
          text-decoration: underline;
        }
      }
    }
    .form {
      margin-bottom: 44px;
      .phone,
      .pwd,
      .code {
        input {
          padding: 0;
          height: 38px;
          border: none;
          border-radius: 0;
          border-bottom: 1px solid #e5e9f2;
          padding-left: 20px;
          font-size: 22px;
          &:focus {
            box-shadow: none;
            border-bottom: 1px solid #ff4949;
          }
        }
      }
      .ivu-form-item {
        margin-bottom: 50px;
        &:last-child {
          margin-bottom: 0;
        }
        .ivu-form-item-content {
          position: relative;
          .icon {
            display: block;
            position: absolute;
            right: 0;
            top: 7px;
            width: 24px;
            height: 24px;
            z-index: 10;
            cursor: pointer;
            background: url(../../assets/image/pwd.png) no-repeat;
            &.active {
              background: url(../../assets/image/pwd_d.png) no-repeat;
            }
          }
          .get-code {
            display: block;
            position: absolute;
            right: 0;
            top: -7px;
            width: 88px;
            height: 36px;
            border-radius: 2px;
            border: solid 1px #ff4949;
            outline: none;
            background: transparent;
            color: #ff4949;
            font-size: 14px;
            z-index: 10;
            cursor: pointer;
            &:hover {
              opacity: .9;
            }
          }
        }
      }
    }
    .foot-btn {
      .checkbox {
        display: block;
        float: left;
      }
      .submit,
      .forget {
        display: block;
        float: right;
      }
      .checkbox {
        display: block;
        width: 80px;
        height: 36px;
        text-align: center;
        line-height: 36px;
        font-size: 14px;
        color: #1f2d3d;
        &:hover {
          color: #ff4949;
        }
      }
      .forget {
        display: block;
        width: 80px;
        height: 36px;
        text-align: center;
        line-height: 36px;
        font-size: 16px;
        color: #1f2d3d;
        text-decoration: none;
        cursor: pointer;
        &:hover {
          color: #ff4949;
        }
      }
      .submit {
        margin-left: 20px;
        width: 144px;
        height: 36px;
        border-radius: 2px;
        background-color: #ff4949;
        border: none;
        outline: none;
        color: #fff;
        font-size: 14px;
        cursor: pointer;
        &:hover {
          opacity: .9;
        }
      }
    }
  }
}
</style>
