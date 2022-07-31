<template>
  <div class="login-container">
    <el-form
        ref="changePassForm"
        :model="formResetPass"
        :rules="activeRules"
        autocomplete="on"
        class="login-form"
        label-position="left"
    >
      <el-card>
        <div class="title-container">
          <img
              alt="ico"
              height="100"
              :src="publicPath"
              width="100"
          >
          <h3 class="title">{{ $t('login.resetPassLabel').toUpperCase() }}</h3>
          <el-tooltip
              :content="$t('login.changeLang')"
              placement="top"
          >
            <lang-select :i18n="i18n" :store="store" class="set-language"/>
          </el-tooltip>
        </div>

        <el-form-item prop="username" style="background: #ffffff;border: 1px solid #e5e5e5;">
          <span class="svg-container">
            <svg-icon icon-class="user"/>
          </span>
          <el-input
              ref="username"
              v-model="formResetPass.username"
              :maxlength="25"
              disabled
              name="username"
              readonly
              tabindex="1"
              type="text"
          />
        </el-form-item>

        <el-tooltip v-model="capsTooltip" :content="$t('login.checkCaplock')" manual placement="right">
          <el-form-item prop="newPassword">
            <span class="svg-container">
              <svg-icon icon-class="password"/>
            </span>
            <el-input
                :key="passwordType"
                ref="newPassword"
                v-model="formResetPass.newPassword"
                :maxlength="50"
                :placeholder="$t('login.newPassword')"
                :type="passwordType"
                autocomplete="on"
                name="password"
                tabindex="2"
                @blur="capsTooltip = false"
                @keyup.native="checkCapslock"
                @keyup.enter.native="onChangePass"
            />
            <span class="show-pwd" @click="showPwd('newPassword')">
              <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'"/>
            </span>
          </el-form-item>
        </el-tooltip>

        <el-form-item prop="confirmPassword">
          <span class="svg-container">
            <svg-icon icon-class="password"/>
          </span>
          <el-input
              :key="confirmPasswordType"
              ref="confirmPassword"
              v-model="formResetPass.confirmPassword"
              :maxlength="50"
              :placeholder="$t('login.confirmPassword')"
              :type="confirmPasswordType"
              autocomplete="on"
              name="confirmPassword"
              tabindex="2"
              @blur="capsTooltip = false"
              @keyup.native="checkCapslock"
              @keyup.enter.native="onChangePass"
          />
          <span class="show-pwd" @click="showConfirmPass('confirmPassword')">
            <svg-icon :icon-class="confirmPasswordType === 'password' ? 'eye' : 'eye-open'"/>
          </span>
        </el-form-item>

        <el-button
            :loading="loading"
            style="width:100%;margin-bottom:30px;"
            tabindex="3"
            type="primary"
            @click.native.prevent="onChangePass"
        >
          {{ $t('login.resetPassLabel') }}
        </el-button>

        <rule-change-pass/>
      </el-card>
    </el-form>
  </div>
</template>

<script>
import {errAlert, showAlert, SUCCESS} from 'ff24-js/src/utils/AlertMessage';
import LangSelect from '../../components/LangSelect/index.vue';
import apiFactory from 'ff24-js/src/api/apiFactory';
import RuleChangePass from './../BaseFormCustoms/RuleChangePass.vue';
import {removeToken} from 'ff24-js/src/utils/authCookie';
import ConstantAPI from 'ff24-js/src/utils/ConstantAPI';
import {validateConfirmPass, validateNewPass} from "ff24-js/src/utils/LoginValidate";

export default {
  name: 'AccountActive',
  components: {LangSelect, RuleChangePass},
  data() {
    return {
      formResetPass: {
        username: atob(this.route.query.username),
        newPassword: '',
        confirmPassword: '',
        oldPassword: 'Asdqwe@124',
        totpKey: this.route.query.token
      },
      activeRules: {
        newPassword: [{required: true, validator: validateNewPass, trigger: 'change'}],
        confirmPassword: [{required: true, validator: validateConfirmPass, trigger: 'change'}]
      },
      passwordType: 'password',
      confirmPasswordType: 'password',
      capsTooltip: false,
      loading: false,
      showDialog: false,
      redirect: undefined,
      otherQuery: {},
      publicPath: `${process.env.VUE_APP_FE_LIB || '/gateway/fe-lib/'}assets/images/logo.png`
    };
  },
  props: {
    store: {
      type: Object,
      required: true,
      default: () => {
      }
    },
    route: {
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
  mounted() {
    if (this.$refs.newPassword && this.formResetPass.newPassword === '') {
      this.$refs.newPassword.focus();
    } else if (this.$refs.confirmPassword && this.formResetPass.confirmPassword === '') {
      this.$refs.confirmPassword.focus();
    }
  },
  methods: {
    onChangePass() {
      this.$refs.changePassForm.validate((valid) => {
        if (!valid) {
          return false;
        }
        apiFactory.callAPI(ConstantAPI.ACCOUNT_CHANGE.RESET_PASSWORD, this.formResetPass).then(() => {
          removeToken()
          this.router.push('/login?redirect=/dashboard');
          showAlert(this, SUCCESS, `Đặt lại mật khẩu thành công : Tài khoản [${this.formResetPass.username}]`);
        }).catch(err => {
          errAlert(this, err);
        });
      });
    },

    checkCapslock(e) {
      const {key} = e;
      this.capsTooltip = key && key.length === 1 && (key >= 'A' && key <= 'Z');
    },

    showPwd(prop) {
      if (this.passwordType === 'password') {
        this.passwordType = '';
      } else {
        this.passwordType = 'password';
      }
      this.$nextTick(() => {
        this.$refs[prop].focus();
      });
    },

    showConfirmPass(prop) {
      if (this.confirmPasswordType === 'password') {
        this.confirmPasswordType = '';
      } else {
        this.confirmPasswordType = 'password';
      }
      this.$nextTick(() => {
        this.$refs[prop].focus();
      });
    },

    getOtherQuery(query) {
      return Object.keys(query).reduce((acc, cur) => {
        if (cur !== 'redirect') {
          acc[cur] = query[cur]
        }
        return acc
      }, {})
    }
  }
};
</script>

<style lang="scss">
$bg: #283443;
$light_gray: #fff;
$cursor: #1c1c1c;
$disableBg: #ffffff;

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

    input:disabled {
      background: $disableBg;
      color: #043244;
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
    padding: 30px 35px 0;
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
