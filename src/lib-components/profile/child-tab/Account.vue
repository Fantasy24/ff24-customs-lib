<template>
  <el-form
      ref="formChangePass"
      :model="formChangePass"
      :rules="ruleChangePass"
      label-width="155px"
  >
    <el-row>
      <el-col :lg="12" :md="16" :sm="24" :xl="12" :xs="24">
        <el-form-item label="Mật khẩu cũ" prop="oldPassword" required>
          <el-input
              v-model="formChangePass.oldPassword"
              :maxlength="50"
              :type="passwordType"
              autocomplete="off"
              clearable
              placeholder="Mật khẩu cũ"
              show-word-limit
              @paste.native.prevent
              @copy.native.prevent
          >
            <span slot="suffix" class="show-pwd">
              <el-button type="text" @click="showPwd('passwordType')">
                <svg-icon
                    :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'"
                />
              </el-button>
            </span>
          </el-input>
        </el-form-item>
      </el-col>
    </el-row>

    <el-row>
      <el-col :lg="12" :md="16" :sm="24" :xl="12" :xs="24">
        <el-form-item label="Mật khẩu mới" prop="newPassword" required>
          <el-input
              v-model="formChangePass.newPassword"
              :maxlength="50"
              :type="newPass"
              autocomplete="off"
              clearable
              placeholder="Mật khẩu mới"
              show-word-limit
              @paste.native.prevent
              @copy.native.prevent
          >
            <span slot="suffix" class="show-pwd">
              <el-button type="text" @click="showPwd('newPass')">
                <svg-icon
                    :icon-class="newPass === 'password' ? 'eye' : 'eye-open'"
                />
              </el-button>
            </span>
          </el-input>
        </el-form-item>
      </el-col>
    </el-row>

    <el-row>
      <el-col :lg="12" :md="16" :sm="24" :xl="12" :xs="24">
        <el-form-item label="Xác nhận mật khẩu" prop="confirmPassword" required>
          <el-input
              v-model="formChangePass.confirmPassword"
              :maxlength="50"
              :type="confirmPass"
              autocomplete="off"
              clearable
              placeholder="Nhập lại mật khẩu mới"
              show-word-limit
              @paste.native.prevent
              @copy.native.prevent
          >
            <span slot="suffix" class="show-pwd">
              <el-button type="text" @click="showPwd('confirmPass')">
                <svg-icon
                    :icon-class="confirmPass === 'password' ? 'eye' : 'eye-open'"
                />
              </el-button>
            </span>
          </el-input>
        </el-form-item>
      </el-col>
    </el-row>

    <el-row>
      <el-col :lg="12" :md="16" :sm="24" :xl="12" :xs="24">
        <el-form-item label="Nhập mã OTP" prop="totpKey" required>
          <el-input
              v-model="formChangePass.totpKey"
              :disabled="!$store.getters.otp"
              :maxlength="8"
              clearable
              placeholder="Mã xác thực từ Email/ Google Authentication"
              show-word-limit
              @keypress.native="onPreventChar"
          />
        </el-form-item>
      </el-col>

      <el-col :lg="12" :md="8" :sm="24" :xl="12" :xs="24">
        <el-button
            v-show="!counting"
            :disabled="!$store.getters.otp"
            icon="el-icon-s-promotion"
            style="margin-left: 10px"
            type="warning"
            @click="onSendCodeVerifyToEmail"
        >
          Gửi mã xác thực OTP
        </el-button>
      </el-col>
    </el-row>

    <el-row v-if="!$store.getters.otp">
      <el-col :lg="16" :md="16" :sm="24" :xl="16" :xs="24">
        <el-form-item>
          <el-alert :closable="false" type="warning">
            Vui lòng
            <el-link style="font-size: 12px" type="primary" @click="syncActiveTab = 'twoFactor'">Kích hoạt xác thực hai
              yếu tố OTP
            </el-link>
            để nhận mã OTP trước khi đổi mật khẩu
          </el-alert>
        </el-form-item>
      </el-col>
    </el-row>

    <el-row v-if="counting" :gutter="20">
      <el-col :lg="12" :md="12" :sm="24" :xl="12" :xs="24">
        <el-form-item label=" ">
          <el-alert :closable="false" type="error">
            Mã xác thực chỉ còn hiệu lực trong
            <countdown :time="REFRESH_TOKEN_TIME" @end="counting = false">
              <template slot-scope="props">{{ props.totalSeconds }}</template>
            </countdown>
            giây
          </el-alert>
        </el-form-item>
      </el-col>
    </el-row>

    <el-form-item>
      <el-button :loading="loading" icon="el-icon-check" type="primary" @click="onChangePass">
        Đổi mật khẩu
      </el-button>
      <el-button icon="el-icon-refresh-left" @click="resetForm">Nhập lại</el-button>
    </el-form-item>

    <rule-change-pass/>
  </el-form>
</template>

<script>
import apiFactory from 'ff24-js/src/api/apiFactory'
import {errAlert, ERROR, showAlert, SUCCESS} from 'ff24-js/src/utils/AlertMessage'
import RuleChangePass from '../../BaseFormCustoms/RuleChangePass.vue'
import ConstantAPI from 'ff24-js/src/utils/ConstantAPI'
import {mapGetters} from 'vuex'
import {REFRESH_TOKEN_TIME} from 'ff24-js/src/utils/Constant'
import countdown from '@chenfengyuan/vue-countdown'
import {mapComputed} from 'ff24-js/src/utils'

export default {
  name: 'Account',
  components: { RuleChangePass, countdown },
  props: {
    activeTab: String
  },
  data() {
    const passLenght = 8
    const validateOldPass = (rule, value, callback) => {
      const reg = [/[a-z]/, /[A-Z]/, /[0-9]/, /[(!@#$%&]/]
      let count = 0
      for (let i = 0; i < reg.length; i++) {
        if (reg[i].test(value)) {
          count++
        }
      }
      if (value === '') {
        callback(new Error('Nhập mật khẩu cũ'))
      } else if (value.length < passLenght) {
        callback(new Error(`Tối thiểu ${passLenght} ký tự`))
      } else if (count < 3) {
        callback(new Error('Mật khẩu không thỏa mãn điều kiện'))
      } else {
        callback()
      }
    }

    const validateNewPass = (rule, value, callback) => {
      const reg = [/[a-z]/, /[A-Z]/, /[0-9]/, /[(!@#$%&]/]
      let count = 0
      for (let i = 0; i < reg.length; i++) {
        if (reg[i].test(value)) {
          count++
        }
      }
      if (value === '') {
        callback(new Error('Nhập mật khẩu mới'))
      } else if (value.length < passLenght) {
        callback(new Error(`Tối thiểu ${passLenght} ký tự`))
      } else if (value.toLowerCase().includes(this.user.uid.toLowerCase())) {
        callback(new Error('[Mật khẩu mới] không được có phần trùng với [Tên tài khoản]'))
      } else if (count < 3) {
        callback(new Error('Mật khẩu không thỏa mãn điều kiện'))
      } else {
        callback()
      }
    }

    const validateConfirmPass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('Nhập lại mật khẩu mới'))
      } else if (value !== this.formChangePass.newPassword) {
        callback(new Error('[Mật khẩu mới] và [Xác nhận mật khẩu] không khớp!'))
      } else {
        callback()
      }
    }
    return {
      REFRESH_TOKEN_TIME,
      formChangePass: {
        oldPassword: '',
        newPassword: '',
        confirmPassword: '',
        username: '',
        totpKey: ''
      },
      passwordType: 'password',
      newPass: 'password',
      confirmPass: 'password',
      counting: false,
      ruleChangePass: {
        oldPassword: [
          { validator: validateOldPass }
        ],
        newPassword: [
          { validator: validateNewPass }
        ],
        confirmPassword: [
          { validator: validateConfirmPass, trigger: ['blur', 'change'] }
        ],
        totpKey: [
          { required: true, message: 'Mã xác thực TOTP bắt buộc nhập' },
          { min: 6, max: 8, message: 'Mã xác thực TOTP gồm 6 ký tự' }
        ]
      },
      loading: false,
      MENU_CODE_API: 'QL-NGUOI-DUNG'
    }
  },
  computed: {
    ...mapGetters(['userInfo']),
    syncActiveTab: mapComputed('activeTab')
  },
  methods: {
    onChangePass() {
      this.$refs['formChangePass'].validate(valid => {
        if (!valid) return false
        this.loading = true
        this.formChangePass.username = this.user.uid
        apiFactory.callAPI(ConstantAPI.ACCOUNT_CHANGE.CHANGE_PASSWORD, this.formChangePass).then(async _ => {
          showAlert(this, SUCCESS, 'Đổi mật khẩu thành công!')
          await this.$store.dispatch('user/logout')
          await this.$router.push(`/login?redirect=/dashboard`)
        }).catch(err => {
          if (err.response.data.status === 400 && err.response.data.name.includes('must be')) {
            return this.$message({
              message: '[TOTP] không hợp lệ',
              type: ERROR
            })
          }
          if (err.response.data.status === 400 && err.response.data.code.includes('EQUALFIELD')) {
            return this.$message({
              message: '[Mật khẩu mới] không được trùng với [Mật khẩu cũ]',
              type: ERROR
            })
          }
          errAlert(this, err)
        }).finally(() => {
          this.loading = false
        })
      })
    },

    onPreventChar($event) {
      const keyCode = ($event.keyCode ? $event.keyCode : $event.which)
      if ((keyCode < 48 || keyCode > 57)) {
        $event.preventDefault()
      }
    },

    onSendCodeVerifyToEmail() {
      apiFactory.callAPI(ConstantAPI['ql-nguoi-dung'].GET_RESEND_TOTP_KEY).then(() => {
        this.counting = true
        showAlert(this, SUCCESS, 'Đã gửi mã xác thực, vui lòng kiểm tra email của bạn')
      }).catch(err => {
        errAlert(this, err)
      })
    },

    showPwd(fieldName) {
      if (this[fieldName] === 'password') {
        this[fieldName] = ''
      } else {
        this[fieldName] = 'password'
      }
    },

    resetForm() {
      this.$refs['formChangePass'].resetFields()
    }
  }
}
</script>
