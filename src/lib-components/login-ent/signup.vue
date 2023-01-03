<template>
  <div class="login-container">
    <el-form
      ref="signUpForm"
      :model="signUpForm"
      :rules="signUpRule"
      class="login-form"
      label-position="left"
    >
      <el-card>
        <div class="title-container">
          <h3 class="title">ĐĂNG KÝ HUNRE</h3>
          <!-- <el-tooltip :content="$t('login.changeLang')" placement="top">
            <lang-select :i18n="i18n" :store="store" class="set-language" />
          </el-tooltip> -->
        </div>

        <!-- <el-form-item prop="code">
          <span class="svg-container">
           <svg-icon icon-class="suitcase"/>
          </span>
        <el-input
            id="code"
            ref="code"
            v-model="signUpForm.code"
            :maxlength="17"
            :placeholder="$t('signUp.taxCode')"
            auto-complete="on"
            name="code"
            tabindex="1"
            type="text"
            @keypress.native="onPreventChar"
            @keyup.enter.native="onSignUp"
        />
      </el-form-item> -->

        <el-form-item prop="code">
          <span class="svg-container">
            <svg-icon icon-class="user" />
          </span>
          <el-input
            id="username"
            v-model="signUpForm.code"
            placeholder="Tên đăng nhập"
          />
        </el-form-item>

        <!--  password    -->
        <el-tooltip
          v-model="capsTooltip"
          :content="$t('login.checkCaplock')"
          manual
          placement="right"
        >
          <el-form-item prop="password">
            <span class="svg-container">
              <svg-icon icon-class="password" />
            </span>
            <el-input
              id="password"
              :key="passwordType"
              ref="password"
              v-model="signUpForm.password"
              :maxlength="50"
              placeholder="Mật khẩu"
              :type="passwordType"
              autocomplete="on"
              name="password"
              tabindex="2"
              @blur="capsTooltip = false"
              @input="
                (v) => {
                  signUpForm.password = v
                    .replace(/[^\u0000-\u007F]+/g, '')
                    .replace(/\s/g, '');
                }
              "
              @keyup.native="checkCapslock"
              @keyup.enter.native="onSignUp"
            />
            <span class="show-pwd" @click="showPwd">
              <svg-icon
                :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'"
              />
              <el-tooltip effect="light" placement="bottom-end">
                <div slot="content">
                  <rule-change-pass />
                </div>
                <el-icon
                  class="el-icon-warning-outline"
                  style="margin-left: 15px"
                />
              </el-tooltip>
            </span>
          </el-form-item>
        </el-tooltip>

        <el-form-item prop="name">
          <span class="svg-container">
            <svg-icon icon-class="fullname" />
          </span>
          <!-- <i class="el-icon-s-custom"></i> -->
          <el-input
            id="name"
            ref="name"
            v-model="signUpForm.name"
            :maxlength="255"
            placeholder="Họ và tên"
            name="name"
            tabindex="3"
            type="text"
            @keyup.enter.native="onSignUp"
          />
        </el-form-item>

        <!-- <el-form-item prop="address">
          <span class="svg-container">
           <svg-icon icon-class="guide"/>
          </span>
        <el-input
            id="address"
            ref="address"
            v-model="signUpForm.address"
            :maxlength="255"
            :placeholder="$t('signUp.address')"
            name="address"
            tabindex="4"
            type="text"
            @keyup.enter.native="onSignUp"
        />
      </el-form-item> -->

        <el-form-item prop="email">
          <span class="svg-container">
            <svg-icon icon-class="email" />
          </span>
          <el-input
            id="email"
            ref="email"
            v-model="signUpForm.email"
            :maxlength="254"
            placeholder="Email"
            name="email"
            tabindex="5"
            @keyup.enter.native="onSignUp"
          />
        </el-form-item>

        <el-form-item prop="tel">
          <span class="svg-container">
            <svg-icon icon-class="phone" />
          </span>
          <el-input
            id="tel"
            ref="tel"
            v-model="signUpForm.tel"
            :maxlength="15"
            placeholder="Số điện thoại"
            name="tel"
            tabindex="6"
            type="text"
            @keypress.native="onPreventChar"
            @keyup.enter.native="onSignUp"
          />
        </el-form-item>
        <select-master-data
          :is-show-option-all="false"
          :v-model.sync="signUpForm.depCode"
          label="Khoa"
          placeholder="Khoa"
          prop-form="depCode"
          :required="true"
          :disabled="false"
          :rules="ruleLabDepartment"
          :is-filter="true"
          :filter-data="masterTypeLabDepartment"
          @keyup.enter.native="onSignUp"
        />
        <!-- <el-form-item prop="bussDate">
          <span class="svg-container">
           <svg-icon icon-class="form"/>
          </span>
        <el-date-picker
            id="bussDate"
            ref="bussDate"
            v-model="signUpForm.bussDate"
            format="dd/MM/yyyy"
            :placeholder="$t('signUp.bussDate')"
            name="bussDate"
            tabindex="7"
            type="date"
            @keyup.enter.native="onSignUp"
        />
      </el-form-item>

      <el-form-item prop="type">
          <span class="svg-container">
           <svg-icon icon-class="list"/>
          </span>
        <el-select
            id="type"
            ref="type"
            v-model="signUpForm.type"
            :placeholder="$t('signUp.type')"
            :loading="loadType"
            name="bussDate"
            filterable
            tabindex="8"
            @keyup.enter.native="onSignUp"
        >
          <el-option
              v-for="item in listCompanyType"
              :key="item.code"
              :title="`${item.code} - ${item.name}`"
              :label="`${item.code} - ${item.name.substring(0,100)}`"
              :value="item.code"
          >
          </el-option>
        </el-select>
      </el-form-item> -->

        <!-- <el-row :gutter="10">
        <el-col :span="12">
          <el-form-item prop="captchaKey">
              <span class="svg-container">
                <svg-icon icon-class="captcha"/>
              </span>
            <el-input
                id="captchaKey"
                ref="captchaKey"
                v-model="signUpForm.captchaKey"
                :placeholder="$t('login.captcha')"
                maxlength="8"
                tabindex="9"
                @keyup.enter.native="onKeyupCaptcha"
            />
          </el-form-item>
        </el-col>

        <el-col :span="12">
          <img
              id="captchaImg"
              :src="`data:image/png;base64,${codeCaptcha}`"
              :title="$t('login.codeCaptcha')"
              alt="ct"
              style="width: 100%;height: 50px;font-size: 11px"
              @click="refreshCaptcha"
          >
        </el-col>
      </el-row> -->
        <el-button
          id="btnSignUp"
          :loading="loading"
          style="width: 100%; margin-bottom: 30px"
          type="primary"
          tabindex="9"
          @click.native.prevent="onSignUp"
        >
          <span>Đăng ký</span>
        </el-button>

        <div style="float: left">
          <router-link to="/login">
            <el-button
              id="isSignUp"
              size=""
              default
              type="text"
              icon="el-icon-back"
            >
              <span>Đăng nhập</span>
            </el-button>
          </router-link>
        </div>
      </el-card>
    </el-form>
  </div>
</template>

<script>
import {
  errAlert,
  ERROR,
  showAlert,
  SUCCESS,
} from "ff24-js/src/utils/AlertMessage";
import apiFactory from "ff24-js/src/api/apiFactory";
import LangSelect from "../../components/LangSelect/index.vue";
import ConstantAPI from "ff24-js/src/utils/ConstantAPI";
import RuleChangePass from "../BaseFormCustoms/RuleChangePass.vue";
import SelectMasterData from "@/components/CommonComponent/SelectMasterData";
import {
  validateUsername,
  validatePassword,
  validateCaptcha,
} from "ff24-js/src/utils/LoginValidate";

const MASTER_DATA_LAB_DEPARTMENT = "LAB_DEPARTMENT";

export default {
  name: "SignUp",
  components: { LangSelect, RuleChangePass, SelectMasterData },
  data() {
    return {
      codeCaptcha: "",
      masterTypeLabDepartment: MASTER_DATA_LAB_DEPARTMENT,
      signUpForm: {
        address: "",
        bussDate: "",
        captchaKey: "",
        captchaSecret: "",
        code: "",
        email: "",
        name: "",
        password: "",
        tel: "",
        type: "",
      },
      ruleLabDepartment: [this.requiredRule("Khoa")],
      signUpRule: {
        code: [
          { required: true, validator: validateUsername, trigger: "change" },
        ],
        name: [
          {
            required: true,
            message: "Nhập họ và tên",
            trigger: "change",
          },
        ],
        type: [
          {
            required: true,
            message: this.$t("signUp.validType"),
            trigger: "change",
          },
        ],
        address: [
          {
            required: true,
            message: this.$t("signUp.validAddress"),
            trigger: "change",
          },
        ],
        tel: [
          {
            required: true,
            message: "Nhập số điện thoại",
            trigger: "change",
          },
        ],
        bussDate: [
          {
            required: true,
            message: this.$t("signUp.validBussDate"),
            trigger: "change",
          },
        ],
        email: [
          {
            required: true,
            message: "Nhập địa chỉ email",
            trigger: "change",
          },
          {
            type: "email",
            message: this.$t("signUp.validEmail"),
            trigger: ["blur", "change"],
          },
        ],
        password: [
          { required: true, validator: validatePassword, trigger: "change" },
        ],
        captchaKey: [
          { required: true, validator: validateCaptcha, trigger: "change" },
        ],
      },
      passwordType: "password",
      capsTooltip: false,
      loading: false,
      otherQuery: {},
      isSignUp: false,
      listCompanyType: [],
      loadType: false,
      publicPath: process.env.BASE_URL,
    };
  },
  props: {
    store: {
      type: Object,
      required: true,
      default: () => {},
    },
    router: {
      type: Object,
      required: true,
      default: () => {},
    },
    i18n: {
      type: Object,
      required: true,
      default: () => {},
    },
    $t: {
      type: Function,
      required: true,
      default: () => {},
    },
  },
  mounted() {
    // this.refreshCaptcha();
    // this.onGetListCompanyType();
  },
  methods: {
    onSignUp() {
      this.$refs.signUpForm.validate((valid) => {
        if (!valid) {
          this.$refs.code.focus();
          return false;
        }
        const loading = this.$loading({
          lock: true,
          text: "Loading",
          spinner: "el-icon-loading",
          background: "rgba(0, 0, 0, 0.7)",
        });
        //Convert Model
        const signupModel = {};
        signupModel.fullName = this.signUpForm.name;
        signupModel.username = this.signUpForm.code;
        signupModel.password = this.signUpForm.password;
        signupModel.phoneNumber = this.signUpForm.tel;
        signupModel.email = this.signUpForm.email;
        signupModel.isGuest = true;
        signupModel.depCode = this.signUpForm.depCode;

        const contentNotify =
          `Bạn đã đăng ký tài khoản [<b>` +
          signupModel.username +
          `</b>] <p> Chúng tôi đã gửi một email xác nhận vào email của bạn. Bạn cần truy cập vào email của bạn kích hoạt để có thể sử dụng được dịch vụ </p>`;
        apiFactory
          .callAPI(ConstantAPI.SIGNUP.SIGN_UP, signupModel)
          .then((rs) => {
            showAlert(this, SUCCESS, contentNotify, 6000, true);
            const loginForm = {
              username: rs.code,
              password: this.signUpForm.password,
              orgCode: rs.code,
            };
            this.resetForm();
            this.onBackLogin();
            this.$emit("signUpSuccess", loginForm);
            loading.close();
          })
          .catch((err) => {
            loading.close();
            this.refreshCaptcha();
            errAlert(this, err);
          });
      });
    },

    onGetListCompanyType() {
      this.loadType = true;
      apiFactory
        .callAPI(ConstantAPI.SIGNUP.GET_COMPANY_TYPE)
        .then((rs) => {
          this.listCompanyType = rs;
          this.loadType = false;
        })
        .catch(() => {
          this.loadType = false;
          showAlert(this, ERROR, "Lỗi danh sách loại hình doanh nghiệp");
        });
    },

    refreshCaptcha() {
      apiFactory
        .callAPI(ConstantAPI.ACCOUNT_CHANGE.GET_CAPTCHA)
        .then((rs) => {
          this.codeCaptcha = rs["captchaImage"];
          this.signUpForm.captchaSecret = rs.captchaSecret;
        })
        .catch((err) => {
          errAlert(this, err);
        });
    },

    onKeyupCaptcha() {
      this.onSignUp();
    },

    checkCapslock(e) {
      const { key } = e;
      this.capsTooltip = key && key.length === 1 && key >= "A" && key <= "Z";
    },

    showPwd() {
      if (this.passwordType === "password") {
        this.passwordType = "";
      } else {
        this.passwordType = "password";
      }
      this.$nextTick(() => {
        this.$refs.password.focus();
      });
    },

    onPreventChar($event) {
      const keyCode = $event.keyCode ? $event.keyCode : $event.which;
      if (keyCode !== 45 && (keyCode < 48 || keyCode > 57)) {
        $event.preventDefault();
      }
    },

    onBackLogin() {
      this.$emit("back", false);
    },

    resetForm() {
      this.$refs.signUpForm.resetFields();
    },
  },
};
</script>

<style lang="scss">
$bg: #283443;
$light_gray: #fff;
$cursor: #1c1c1c;
$disableBg: #e5e5e5;

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
      padding: 15px 0px 0px 30px !important;
      color: #043244;
      height: 30px !important;
      caret-color: $cursor;

      &:-webkit-autofill {
        -webkit-box-shadow: 0 0 0 1000px transparent inset !important;
        box-shadow: inset 0 0 0 1000px #e5e5e5 !important;
        -webkit-text-fill-color: #043244 !important;
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
    height: 45px !important;
    margin-bottom: 20px !important;

    .el-form-item__error--inline {
      top: -8px !important;
    }

    .el-select {
      width: 100%;
      position: absolute;

      .el-input__suffix {
        right: -42px !important;
      }
    }
  }
}
</style>

<style lang="scss" scoped>
$bg: #ffffff;
// $dark_gray: #1682e6;
// $light_gray: #043244;  #17f001;
$dark_gray: #67c23a;
$light_gray: #67c23a;

.login-container {
  min-height: 100%;
  width: 100%;
  background-color: $bg;
  // background-image: linear-gradient(#01aef0, #ffffff);
  background-image: linear-gradient(#67c23a, #ffffff);
  overflow: hidden;

  .login-form {
    position: relative;
    width: 750px;
    max-width: 100%;
    padding: 30px 30px 30px;
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
      /* color: #01aef0; 
        background-color: #01aef0;*/
      color: #67c23a;
      background-color: #67c23a;
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
