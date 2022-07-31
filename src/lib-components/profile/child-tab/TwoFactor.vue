<template>
  <div>
    <el-tabs v-model="activeName" type="border-card" @tab-click="handleClick">
      <el-tab-pane name="email">
        <span slot="label"><el-icon class="el-icon-message"/> Xác thực qua Email</span>
        <div class="EmptyBox110"/>

        <el-alert
          v-if="$store.getters.otp === true && !totpSecret"
          :closable="false"
          show-icon
          title="Bạn đã xác thực hai yếu tố thành công. Nếu muốn thay đổi, vui lòng click vào nút [Gửi mã xác thực OTP] và thực hiện xác thực lại."
          type="success"
        />

        <div class="EmptyBox10"/>

        <el-row :gutter="20">
          <el-form ref="formEmail" :model="formEmail" label-width="120px">
            <el-col :span="8">
              <el-form-item
                label="Email"
                prop="email"
              >
                <span style="font-weight: bold">{{ formEmail.email }}</span>
              </el-form-item>
            </el-col>

            <el-col :span="2">
              <el-form-item>
                <el-button
                  v-show="isShowBtnSend"
                  icon="el-icon-s-promotion"
                  type="primary"
                  @click="onSendCodeVerifyToEmail('formEmail')"
                >
                  Gửi mã xác thực OTP
                </el-button>
              </el-form-item>
            </el-col>
          </el-form>
        </el-row>
        <div class="EmptyBox20"/>
        <el-row>
          <el-form ref="formVerify" :model="formVerify" :rules="rule" label-width="120px">
            <el-row v-if="!isShowBtnSend" :gutter="20">
              <el-col :span="8">
                <el-alert v-show="counting" :closable="false" style="width: 100%;" type="error">
                  Mã xác thực chỉ còn hiệu lực trong
                  <countdown v-if="counting" :time="REFRESH_TOKEN_TIME" @end="handleCountdownEnd">
                    <template slot-scope="props">{{ props.totalSeconds }}</template>
                  </countdown>
                  giây
                </el-alert>
              </el-col>
              <el-col :span="2">
                <el-form-item>
                  <el-button
                    v-show="!counting"
                    icon="el-icon-refresh-left"
                    type="warning"
                    @click="onSendCodeVerifyToEmail('formEmail')"
                  >
                    Gửi lại mã xác thực OTP
                  </el-button>
                </el-form-item>
              </el-col>
            </el-row>

            <div class="EmptyBox20"/>

            <el-row v-if="totpSecret" :gutter="20">
              <el-col :span="10">
                <el-form-item
                  label="Mã xác thực"
                  prop="code"
                >
                  <el-input
                    v-model="formVerify.code"
                    :maxlength="8"
                    clearable
                    placeholder="Mã xác thực từ Email/ Google Authentication"
                    show-word-limit
                    @keypress.native="onPreventChar"
                  />
                </el-form-item>
              </el-col>

              <el-col :span="2">
                <el-form-item>
                  <el-button
                    :disabled="formVerify.code === ''"
                    icon="el-icon-finished"
                    type="success"
                    @click="onVerifyCode('formVerify')"
                  >
                    Xác thực mã và tiếp tục
                  </el-button>
                </el-form-item>
              </el-col>
            </el-row>
          </el-form>
        </el-row>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import apiFactory from 'ff24-js/src/api/apiFactory'
import {errAlert, ERROR, showAlert, SUCCESS} from 'ff24-js/src/utils/AlertMessage'
import countdown from '@chenfengyuan/vue-countdown'
import {REFRESH_TOKEN_TIME} from 'ff24-js/src/utils/Constant'
import ConstantAPI from 'ff24-js/src/utils/ConstantAPI'

export default {
  name: 'TwoFactor',
  components: {
    countdown
  },
  data() {
    return {
      REFRESH_TOKEN_TIME,
      isShowBtnSend: true,
      formEmail: {
        email: ''
      },
      formVerify: {
        code: null
      },
      formQr: {
        code: ''
      },
      rule: {
        code: [
          { min: 6, max: 8, message: 'Mã xác thực gồm 6 hoặc 8 số', trigger: ['blur', 'change'] }
        ]
      },
      activeName: 'email',
      totpSecret: '',
      isShowSecretCode: false,
      loading: false,
      counting: false
    }
  },
  computed: {
    user() {
      return this.$store.getters.userInfo
    }
  },
  mounted() {
    this.formEmail.email = this.user.ema
  },
  methods: {
    handleClick() {
      this.activeName = 'email'
      this.$refs.formEmail.resetFields()
      this.$refs.formVerify.resetFields()
      this.$refs.formQr.resetFields()
    },

    onSendCodeVerifyToEmail(formName) {
      this.$refs[formName].validate((valid) => {
        if (!valid) {
          return false
        }
        apiFactory.callAPI(ConstantAPI['ql-nguoi-dung'].GET_TOTP_SECRET).then(rs => {
          if (rs) {
            this.totpSecret = rs
            this.formVerify.code = ''
            this.isShowBtnSend = false
            this.counting = true
          }
        }).catch(err => {
          errAlert(this, err)
          this.formVerify.code = ''
        })
      })
    },

    handleCountdownEnd() {
      this.counting = false
    },

    onVerifyCode(formName) {
      this.$refs[formName].validate((valid) => {
        if (!valid) {
          return false
        }
        const payload = {
          totpKey: this.formQr.code || this.formVerify.code,
          totpSecret: this.totpSecret
        }
        apiFactory.callAPI(ConstantAPI['ql-nguoi-dung'].UPDATE_TOTP_SECRET, payload).then(() => {
          this.counting = false
          this.isShowBtnSend = true
          this.totpSecret = ''
          this.resetForm('formVerify')
          this.$store.dispatch('user/logout')
          this.$router.push(`/login?redirect=/dashboard`)
          showAlert(this, SUCCESS, 'Kích hoạt xác thực hai yếu tố OTP thành công. Vui lòng đăng nhập lại để sử dụng.', 5000)
        }).catch(err => {
          if (err.response.data.status === 500 && err.response.data.message.includes('string')) {
            this.formVerify.code = ''
            return this.$message({
              message: '[Mã xác thực] không hợp lệ',
              type: ERROR
            })
          }
          errAlert(this, err)
          this.formVerify.code = ''
        })
      })
    },

    onPreventChar($event) {
      const keyCode = ($event.keyCode ? $event.keyCode : $event.which)
      if ((keyCode < 48 || keyCode > 57)) {
        $event.preventDefault()
      }
    },

    resetForm(formName) {
      this.$refs[formName].resetFields()
    }
  }
}
</script>
