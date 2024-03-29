<template>
  <div class="login-container">
    <div v-if="!isSignUp">
      <el-form
          v-if="!isRenderFormOtp"
          ref="loginForm"
          :model="loginForm"
          :rules="loginRules"
          autocomplete="on"
          class="login-form"
          label-position="left"
      >
        <el-card>
          <div class="title-container">
            <h3 class="title">{{ $t('login.logIn') }}</h3>
            <el-tooltip
                :content="$t('login.changeLang')"
                placement="top"
            >
              <lang-select :i18n="i18n" :store="store" class="set-language"/>
            </el-tooltip>
          </div>

          <el-form-item prop="orgCode">
          <span class="svg-container">
            <svg-icon icon-class="suitcase"/>
          </span>
            <el-input
                id="orgCode"
                ref="orgCode"
                v-model="loginForm.orgCode"
                :maxlength="17"
                :placeholder="$t('signUp.taxCode')"
                auto-complete="on"
                name="orgCode"
                tabindex="1"
                type="text"
                @keyup.enter.native="onLogin"
            />
          </el-form-item>

          <el-form-item prop="username">
          <span class="svg-container">
            <svg-icon icon-class="user"/>
          </span>
            <el-input
                id="username"
                ref="username"
                v-model="loginForm.username"
                :maxlength="25"
                :placeholder="$t('login.username')"
                auto-complete="on"
                name="username"
                tabindex="2"
                type="text"
                @keyup.enter.native="onLogin"
            />
          </el-form-item>

          <el-tooltip
              v-if="!isForgotPass"
              v-model="capsTooltip"
              :content="$t('login.checkCaplock')"
              manual
              placement="right"
          >
            <el-form-item prop="password">
            <span class="svg-container">
              <svg-icon icon-class="password"/>
            </span>
              <el-input
                  id="password"
                  :key="passwordType"
                  ref="password"
                  v-model="loginForm.password"
                  :maxlength="50"
                  :placeholder="$t('login.password')"
                  :type="passwordType"
                  autocomplete="on"
                  name="password"
                  tabindex="3"
                  @blur="capsTooltip = false"
                  @input="v => {
                  loginForm.password = v.replace(/[^\u0000-\u007F]+/g, '').replace(/\s/g, '');
                }"
                  @keyup.native="checkCapslock"
                  @keyup.enter.native="onLogin"
              />
              <span class="show-pwd" @click="showPwd">
              <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'"/>
            </span>
            </el-form-item>
          </el-tooltip>

          <el-row :gutter="10">
            <el-col :xs="24" :sm="24" :md="12" :lg="12" :xl="12">
              <el-form-item prop="captchaKey">
              <span class="svg-container">
                <svg-icon icon-class="captcha"/>
              </span>
                <el-input
                    id="captchaKey"
                    ref="captchaKey"
                    v-model="loginForm.captchaKey"
                    :placeholder="$t('login.captcha')"
                    maxlength="8"
                    tabindex="4"
                    @keyup.enter.native="onKeyupCaptcha"
                />
              </el-form-item>
            </el-col>
            <el-col :xs="24" :sm="24" :md="12" :lg="12" :xl="12">
              <img
                  v-if="codeCaptcha"
                  id="captchaImg"
                  :src="`data:image/png;base64,${codeCaptcha}`"
                  :title="$t('login.codeCaptcha')"
                  alt="ct"
                  style="width: 100%;height: 50px;font-size: 11px"
                  @click="refreshCaptcha"
              >

              <div
                  v-else
                  style="text-align:center;border: 1px dotted;padding: 0.9em;color: #ec5656;cursor: pointer"
                  @click="location.reload()"
              >
                SERVER ERROR
              </div>
            </el-col>
          </el-row>
          <div style="height: 10px"/>
          <el-button
              v-if="!isForgotPass"
              id="btnLogin"
              :loading="loading"
              style="width:100%;margin-bottom:30px;"
              tabindex="5"
              type="primary"
              @click.native.prevent="onLogin"
          >
            <span>{{ $t('login.logIn') }}</span>
          </el-button>

          <el-button
              v-if="isForgotPass"
              :loading="loading"
              :disabled="!loginForm.orgCode || !loginForm.username || !loginForm.captchaKey"
              id="btnSendForgotPass"
              style="width:100%;margin-bottom:30px;"
              tabindex="3"
              type="primary"
              @click.native.prevent="onSendForgotPass"
          >
            <span>{{ $t('login.forgotPass') }}</span>
          </el-button>

          <div v-if="isClient" style="float: left">
            <el-button id="isSignUp" size="default" type="text" @click="onClickSignUp">
              <strong style="color: #67c23a !important;">{{ $t('signUp.signUp') }}</strong>
            </el-button>
          </div>

          <div style="float: right">
            <el-button id="btnForgotPass" size="medium" type="text" @click="onForgotPass">
              <span v-if="!isForgotPass">{{ $t('login.forgotPassLabel') }}</span>
              <span v-else>{{ $t('login.backToLogin') }}</span>
            </el-button>
          </div>
        </el-card>
      </el-form>

      <el-form
          v-if="isRenderFormOtp"
          ref="otpForm"
          :model="otpForm"
          :rules="otpRules"
          class="login-form"
      >
        <el-card>
          <div class="title-container">
            <h3 class="title">{{ $t('login.twoFactorHeader') }}</h3>
            <el-tooltip
                :content="$t('login.changeLang')"
                placement="top"
            >
              <lang-select :i18n="i18n" :store="store" class="set-language"/>
            </el-tooltip>
          </div>

          <el-row :gutter="20">
            <el-col :span="24">
              <el-alert v-show="counting" :closable="false" style="width: 100%;" type="error">
                {{ $t('login.countdownOtp') }}
                <countdown v-if="counting" :time="REFRESH_TOKEN_TIME" @end="handleCountdownEnd">
                  <template slot-scope="props">{{ props.totalSeconds }}</template>
                </countdown>
                {{ $t('login.second') }}
              </el-alert>
            </el-col>
          </el-row>
          <div class="EmptyBox20"/>
          <el-form-item prop="totpKey">
          <span class="svg-container">
            <em class="el-icon-key"/>
          </span>
            <el-input
                id="totpKey"
                ref="totpKey"
                v-model="otpForm.totpKey"
                :maxlength="8"
                :placeholder="$t('login.placeholderOtp')"
                name="totpKey"
                tabindex="1"
                @keypress.native="onPreventChar"
                @keyup.enter.native="onLoginWithOtp"
                @input="v => {otpForm.totpKey = v.replace(/\D/g,'')}"
            />
          </el-form-item>

          <div style="text-align: center">
            <el-button
                id="btnLoginOtp"
                :loading="loading"
                style="margin-bottom:30px"
                tabindex="2"
                type="primary"
                @click.native.prevent="onLoginWithOtp"
            >
              <span>{{ $t('login.logIn') }}</span>
            </el-button>

            <el-button
                v-show="!counting"
                icon="el-icon-refresh"
                type="text"
                style="color: red"
                @click="onExpireTotp"
            >
              {{ $t('login.expireOtp') }}
            </el-button>
          </div>
        </el-card>
      </el-form>
    </div>

    <div v-if="isSignUp">
      <sign-up
          :$t="$t"
          :i18n="i18n"
          :router="router"
          :store="store"
          @back="isSignUp = false"
          @signUpSuccess="onSignUpSuccess"
      />
    </div>
  </div>
</template>

<script>
import {errAlert, showAlert, SUCCESS} from 'ff24-js/src/utils/AlertMessage';
import SignUp from './signup.vue';
import apiFactory from 'ff24-js/src/api/apiFactory';
import LangSelect from '../../components/LangSelect/index.vue';
import countdown from '@chenfengyuan/vue-countdown';
import {setToken} from 'ff24-js/src/utils/authCookie';
import ConstantAPI from 'ff24-js/src/utils/ConstantAPI';
import {REFRESH_TOKEN_TIME} from 'ff24-js/src/utils/Constant';
import {validateCaptcha, validateOtpKey, validatePassword, validateUsername, validateOrgCode} from "ff24-js/src/utils/LoginValidate";

export default {
  name: 'LoginEnt',
  components: { LangSelect, SignUp, countdown },
  data() {
    return {
      REFRESH_TOKEN_TIME,
      location,
      codeCaptcha: '',
      isShowBtnSend: true,
      counting: true,
      isRenderFormOtp: false,
      loginForm: {
        username: '',
        password: '',
        captchaSecret: '',
        captchaKey: '',
        orgCode: ''
      },
      otpForm: {
        accessToken: '',
        tokenType: 'otp',
        totpKey: ''
      },
      otpRules: {
        totpKey: [{ required: true, validator: validateOtpKey, trigger: 'change' }]
      },
      loginRules: {
        orgCode: [{ required: true, validator: validateOrgCode, trigger: 'change' }],
        username: [{ required: true, validator: validateUsername, trigger: 'change' }],
        password: [{ required: true, validator: validatePassword, trigger: 'change' }],
        captchaKey: [{ required: true, validator: validateCaptcha, trigger: 'change' }]
      },
      passwordType: 'password',
      capsTooltip: false,
      loading: false,
      showDialog: false,
      redirect: undefined,
      otherQuery: {},
      isForgotPass: false,
      isSignUp: false,
      isClient: process.env.VUE_APP_ENDPOINT_AUTH,
      publicPath: process.env.BASE_URL
    }
  },
  props: {
    store: {
      type: Object,
      required: true,
      default: () => {
      }
    },
    router: {
      type: Object,
      required: true,
      default: () => {
      }
    },
    i18n: {
      type: Object,
      required: true,
      default: () => {
      }
    },
    $t: {
      type: Function,
      required: true,
      default: () => {
      }
    }
  },
  watch: {
    router: {
      handler: function(route) {
        const query = route.query
        if (query) {
          this.redirect = query.redirect
          this.otherQuery = this.getOtherQuery(query)
        }
      },
      immediate: true
    }
  },
  mounted() {
    this.refreshCaptcha()
    if (this.$refs.orgCode && this.loginForm.orgCode === '') {
      this.$refs.orgCode.focus()
    } else if (this.$refs.username && this.loginForm.username === '') {
      this.$refs.username.focus()
    } else if (this.$refs.password && this.loginForm.password === '') {
      this.$refs.password.focus()
    }
  },
  methods: {
    refreshCaptcha() {
      apiFactory.callAPI(ConstantAPI.ACCOUNT_CHANGE.GET_CAPTCHA).then(rs => {
        this.codeCaptcha = rs['captchaImage']
        this.loginForm.captchaSecret = rs.captchaSecret
      }).catch(err => {
        errAlert(this, err)
      })
    },

    onKeyupCaptcha() {
      if (!this.isForgotPass) {
        this.onLogin()
      } else {
        this.onSendForgotPass()
        this.resetForm()
      }
    },

    onLogin() {
      this.$refs.loginForm.validate(valid => {
        if (!valid) {
          return false
        }
        const loading = this.$loading({
          lock: true,
          text: 'Loading',
          spinner: 'el-icon-loading',
          background: 'rgba(0, 0, 0, 0.7)'
        })
        this.loading = true
        this.store.dispatch('user/login', this.loginForm).then(rs => {
          if (rs.tokenType === 'otp') {
            this.isRenderFormOtp = true
            this.counting = true
            this.otpForm.accessToken = rs.accessToken
            this.otpForm.totpKey = ''
            loading.close()
          } else {
            setTimeout(() => {
              loading.close()
            }, 100)
            this.router.push({ path: this.redirect || '/', query: this.otherQuery })
            localStorage.clear()
            location.reload()
          }
          this.loading = false
        }).catch((err) => {
          this.loading = false
          loading.close()
          this.refreshCaptcha()
          this.loginForm.captchaKey = ''
          this.$refs.captchaKey.focus()
          errAlert(this, err)
        })
      })
    },

    onLoginWithOtp() {
      this.$refs.otpForm.validate((valid) => {
        if (!valid) {
          return false
        }
        this.loading = true
        apiFactory.callAPI(ConstantAPI.LOGIN.SIGN_IN_OTP, this.otpForm).then(rs => {
          localStorage.clear()
          location.reload()
          this.router.push({ path: this.redirect || '/', query: this.otherQuery })
          this.store.commit('SET_TOKEN', rs.accessToken)
          setToken(rs.accessToken)
          this.loading = false
        }).catch(err => {
          errAlert(this, err)
          this.loading = false
          this.otpForm.totpKey = ''
          if (this.$refs.otpForm && this.otpForm.totpKey === '') {
            this.$refs.totpKey.focus()
          }
        })
      })
    },

    onSignUpSuccess(loginForm){
      this.loginForm = loginForm
      this.refreshCaptcha()
    },

    onExpireTotp() {
      this.isRenderFormOtp = false
      this.loginForm.captchaKey = ''
      this.otpForm.totpKey = ''
      this.refreshCaptcha()
    },

    checkCapslock(e) {
      const { key } = e
      this.capsTooltip = key && key.length === 1 && (key >= 'A' && key <= 'Z')
    },

    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.password.focus()
      })
    },

    getOtherQuery(query) {
      return Object.keys(query).reduce((acc, cur) => {
        if (cur !== 'redirect') {
          acc[cur] = query[cur]
        }
        return acc
      }, {})
    },

    onSendForgotPass() {
      apiFactory.callAPI(ConstantAPI.ACCOUNT_CHANGE.GET_RESET_PASSWORD, {}, this.loginForm).then(() => {
        showAlert(this, SUCCESS, this.$t('login.sendResetPassMsg'))
        this.isForgotPass = false
        this.loginForm.captchaKey = ''
        this.resetForm()
        this.refreshCaptcha()
      }).catch((err) => {
        errAlert(this, err)
        this.refreshCaptcha()
        this.loginForm.captchaKey = ''
        this.$refs.captchaKey.focus()
      })
    },

    onForgotPass() {
      this.isForgotPass = !this.isForgotPass
      this.refreshCaptcha()
      this.resetForm()
    },

    handleCountdownEnd() {
      this.counting = false
    },

    onPreventChar($event) {
      const keyCode = ($event.keyCode ? $event.keyCode : $event.which)
      if ((keyCode < 48 || keyCode > 57)) {
        $event.preventDefault()
      }
    },

    onClickSignUp() {
      this.isSignUp = !this.isSignUp
      this.refreshCaptcha()
      this.resetForm()
    },

    resetForm() {
      this.$refs.loginForm.resetFields()
    }
  }
}
</script>

<style lang="scss">
$bg: #283443;
$light_gray: #fff;
$cursor: #1c1c1c;

@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .login-container .el-input input {
    color: $cursor;
  }
}

/* reset element-ui css */
.login-container {
  .el-input {
    display: inline-block;
    height: 47px;
    width: 85%;

    input {
      background: transparent;
      border: 0;
      -webkit-appearance: none;
      border-radius: 0;
      padding: 12px 5px 12px 15px;
      color: #043244;
      height: 47px;
      caret-color: $cursor;

      &:-webkit-autofill {
        -webkit-box-shadow: 0 0 0 1000px transparent inset !important;
        box-shadow: inset 0 0 0 1000px #e5e5e5 !important;
        -webkit-text-fill-color: #043244 !important
      }
    }
  }

  .el-form-item {
    border: 1px solid white;
    background: rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    color: #454545;
  }
}
</style>

<style lang="scss" scoped>
$bg: #ffffff;
$dark_gray: #1682e6;
$light_gray: #043244;

.login-container {
  min-height: 100%;
  width: 100%;
  background-color: $bg;
  background-image: linear-gradient(#01aef0, #ffffff);
  overflow: hidden;

  .login-form {
    position: relative;
    width: 750px;
    max-width: 100%;
    padding: 160px 35px 0;
    margin: 0 auto;
    overflow: hidden;
  }

  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    text-align: center;
    position: relative;

    .title {
      font-size: 26px;
      color: $light_gray;
      margin: 0 auto 40px auto;
      text-align: center;
      font-weight: bold;
    }

    .set-language {
      color: #01aef0;
      background-color: #01aef0;
      position: absolute;
      top: 3px;
      font-size: 18px;
      right: 0;
      cursor: pointer;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }

  .thirdparty-button {
    position: absolute;
    right: 0;
    bottom: 6px;
  }

  @media only screen and (max-width: 470px) {
    .thirdparty-button {
      display: none;
    }
  }
}
</style>
