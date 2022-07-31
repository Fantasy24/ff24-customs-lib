<template>
  <div class="app-container">
    <el-card shadow="never">
      <div>
        <el-form ref="form" :model="form" :rules="rulesEdit" label-width="190px">
          <el-form-item label="Đơn vị" required style="width: 514px;">
            <span v-if="isHiddenInput">{{ user.org }} - {{ user.orgName }}</span>
            <select-ma-hq
              v-else
              :disabled="true"
              :is-require="true"
              :is-show-option-all="false"
              :rules="rulesEdit.orgCode"
              :s-cusid.sync="form.orgCode"
              prop-form="orgCode"
            />
          </el-form-item>

          <el-row :gutter="10">
            <el-col :span="12">
              <el-form-item label="Tên đăng nhập" prop="username" required>
                <span v-if="isHiddenInput">{{ form.username }}</span>
                <el-input v-else v-model="form.username" disabled/>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="Mã công chức">
                <span v-if="isHiddenInput">{{ form.idCard }}</span>
                <el-input v-else v-model="form.idCard" clearable disabled maxlength="50" show-word-limit/>
              </el-form-item>
            </el-col>
          </el-row>

          <el-form-item label="Họ và tên" prop="fullname" required>
            <span v-if="isHiddenInput">{{ form.fullname }}</span>
            <el-input v-else v-model="form.fullname" clearable maxlength="500" show-word-limit/>
          </el-form-item>

          <el-row :gutter="10">
            <el-col :span="12">
              <el-form-item label="Phòng ban">
                <span v-if="isHiddenInput">{{ user.dep }} - {{ user.depName }}</span>
                <select-dep-code
                  v-else
                  :dep-code.sync="form.depCode"
                  :disabled="true"
                  :store="$store"
                  label=""
                  prop-form="depCode"
                />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="Chức vụ">
                <span v-if="isHiddenInput">{{ user.pos }} - {{ user.posName }}</span>
                <select-pos-code
                  v-else
                  :disabled="true"
                  :pos-code.sync="form.posCode"
                  :store="$store"
                  label=""
                  prop-form="posCode"
                />
              </el-form-item>
            </el-col>
          </el-row>

          <el-row :gutter="10">
            <el-col :span="12">
              <el-form-item label="Ngày sinh">
                <span v-if="isHiddenInput">{{ form.dateOfBirth | formatDate }}</span>
                <el-date-picker
                  v-else
                  v-model="form.dateOfBirth"
                  v-mask="'##-##-####'"
                  :format="DATE.DD_MM_YYYY2"
                  :picker-options="maxDateOfBirth"
                  clearable
                  placeholder="__-__-____"
                  style="width: 100%;"
                  type="date"
                />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="Giới tính" prop="gender">
                <span v-if="isHiddenInput">
                  {{
                    form.gender ? LIST_GENDER.filter(e => e.code === form.gender)[0].name : null
                  }}
                </span>
                <el-select
                  v-else
                  v-model="form.gender"
                  clearable
                  style="width: 100%;"
                >
                  <el-option
                    v-for="gender in LIST_GENDER"
                    :key="gender.code"
                    :label="gender.name"
                    :value="gender.code"
                  />
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>

          <el-row :gutter="10">
            <el-col :lg="12" :md="12" :sm="24" :xl="12" :xs="24">
              <el-form-item label="Số CCCD/CMND" prop="citizenId">
                <span v-if="isHiddenInput">{{ form.citizenId }}</span>
                <el-input-etc
                  v-else
                  v-model="form.citizenId"
                  maxlength="12"
                  placeholder="Số CCCD/CMND"
                  @change="onClearCitizenId"
                  @keypress.native="preventChar"
                />
              </el-form-item>
            </el-col>
            <el-col :lg="12" :md="12" :sm="24" :xl="12" :xs="24">
              <el-form-item
                :rules="form.citizenId ? rulesCitizen.citizenDate : {}"
                label="Ngày cấp CCCD/CMND"
                prop="citizenDate"
              >
                <span v-if="isHiddenInput">
                  {{ form.citizenDate | formatDate }}
                </span>
                <el-date-picker
                  v-else
                  v-model="form.citizenDate"
                  v-mask="'##-##-####'"
                  :format="DATE.DD_MM_YYYY2"
                  :picker-options="maxDateOfBirth"
                  clearable
                  placeholder="__-__-____"
                  style="width: 100%;"
                  type="date"
                  @change="$refs.form.clearValidate('citizenDate')"
                />
              </el-form-item>
            </el-col>
          </el-row>

          <el-form-item
            :rules="form.citizenId ? rulesCitizen.citizenIssuedBy : {}"
            label="Nơi cấp CCCD/CMND"
            prop="citizenIssuedBy"
          >
            <span v-if="isHiddenInput">{{ form.citizenIssuedBy }}</span>
            <el-input-etc
              v-else
              v-model="form.citizenIssuedBy"
              maxlength="255"
              placeholder="Nơi cấp CCCD/CMND"
              @input="$refs.form.clearValidate('citizenIssuedBy')"
            />
          </el-form-item>

          <el-form-item label="Địa chỉ">
            <span
              v-if="isHiddenInput"
              v-html="form.address ? form.address.replaceAll('\n','<br/>') : ''"
            />
            <el-input-etc
              v-else
              v-model="form.address"
              maxlength="500"
              placeholder="Địa chỉ"
              type="textarea"
            />
          </el-form-item>

          <el-row :gutter="10">
            <el-col :lg="12" :md="12" :sm="24" :xl="12" :xs="24">
              <el-form-item label="E-mail" prop="email" required>
                <span v-if="isHiddenInput">{{ form.email }}</span>
                <el-input v-else v-model="form.email" clearable disabled maxlength="255" show-word-limit/>
              </el-form-item>
            </el-col>
            <el-col :lg="12" :md="12" :sm="24" :xl="12" :xs="24">
              <el-form-item
                label="Số điện thoại"
                prop="phoneNumber"
              >
                <span v-if="isHiddenInput">{{ form.phoneNumber }}</span>
                <el-input
                  v-else
                  v-model="form.phoneNumber"
                  :maxlength="11"
                  autocomplete="off"
                  clearable
                  show-word-limit
                />
              </el-form-item>
            </el-col>
          </el-row>

          <el-form-item label="Mô tả">
            <span v-if="isHiddenInput">{{ form.description }}</span>
            <el-input
              v-else
              v-model="form.description"
              clearable
              maxlength="255"
              show-word-limit
              type="textarea"
            />
          </el-form-item>

          <el-form-item v-if="isHiddenInput">
            <el-button
              icon="el-icon-edit"
              style="float: right"
              type="primary"
              @click="onPrepareEdit(false)"
            >
              Sửa
            </el-button>
          </el-form-item>
          <el-form-item v-else style="float: right">
            <el-button icon="el-icon-close" @click="resetForm('form')">Hủy</el-button>
            <el-button
              :loading="loading"
              icon="el-icon-check"
              type="primary"
              @click="onExecuteSave('form')"
            >
              {{ $t('baseLabel.btnSave') }}
            </el-button>
          </el-form-item>
        </el-form>
      </div>
    </el-card>
  </div>
</template>
<script>
import {errAlert, showAlert, SUCCESS} from 'ff24-js/src/utils/AlertMessage'
import apiFactory from 'ff24-js/src/api/apiFactory'
import SelectMaHq from '../../BaseFormCustoms/SelectListMaHq.vue'
import SelectPosCode from '../../BaseFormCustoms/SelectListPosition.vue'
import SelectDepCode from '../../BaseFormCustoms/SelectListDepartment.vue'
import moment from 'moment'
import {DATE, DATE_FORMAT} from 'ff24-js/src/utils/Constant'
import ConstantAPI from 'ff24-js/src/utils/ConstantAPI'
import {numberRule, requiredRule} from 'ff24-js/src/utils'
import {preventChar} from 'ff24-js/src/utils/ECustomsUtils'
import {mask} from 'vue-the-mask'
import ElInputEtc from '../../../components/ElInputETC/ElInputEtc.vue'

const LIST_GENDER = [
  { code: 0, name: 'Không xác định' },
  { code: 1, name: 'Nam' },
  { code: 2, name: 'Nữ' }
]

export default {
  name: 'UserInformation',
  directives: { mask },
  components: {
    ElInputEtc,
    SelectMaHq,
    SelectPosCode,
    SelectDepCode
  },
  filters: {
    formatDate(value) {
      if (value) {
        return moment(String(value)).format(DATE_FORMAT.DD_MM_YYYY)
      }
    }
  },
  data() {
    return {
      loading: false,
      tempData: {},
      LIST_GENDER,
      DATE,
      form: {
        username: '',
        fullname: '',
        orgCode: '',
        dateOfBirth: undefined,
        lastChangePassDt: undefined,
        idCard: '',
        email: '',
        phoneNumber: '',
        depCode: '',
        posCode: '',
        description: '',
        citizenDate: '',
        citizenId: '',
        citizenIssuedBy: '',
        gender: '',
        address: ''
      },
      isHiddenInput: true,
      maxDateOfBirth: {
        disabledDate(date) {
          return date > new Date()
        }
      },
      rulesCitizen: {
        citizenDate: [requiredRule('Ngày cấp CCCD/CMND')],
        citizenIssuedBy: [requiredRule('Nơi cấp CCCD/CMND')]
      },
      rulesEdit: {
        fullname: [requiredRule('Họ và tên')],
        email: [
          requiredRule('Email'),
          { type: 'email', message: 'Email không đúng định dạng' }
        ],
        phoneNumber: [
          { pattern: '([0-9 +\\-.)(_]){10,11}', message: ' Số điện thoại không đúng định dạng' }
        ],
        citizenId: [
          numberRule('Số CCCD/CMND'),
          { min: 9, max: 12, message: 'Số CCCD/CMND từ 9 -> 12 số' }
        ]
      }
    }
  },
  computed: {
    user() {
      return this.$store.getters.userInfo
    }
  },
  mounted() {
    if (this.user && this.user.uid) {
      this.onPrepareEdit(true)
    }
  },
  methods: {
    onPrepareEdit(isDisabledEdit) {
      const params = { username: this.user.uid }
      apiFactory.callAPI(ConstantAPI['ql-nguoi-dung'].SELECT_ITEM, {}, params).then(rs => {
        this.form = rs
        if (isDisabledEdit) {
          this.tempData = rs
        }
        this.isHiddenInput = isDisabledEdit
      }).catch(err => {
        errAlert(this, err)
      })
    },

    onExecuteSave(formName) {
      this.$refs[formName].validate((valid) => {
        if (!valid) return false
        this.loading = true
        apiFactory.callAPI(ConstantAPI['ql-nguoi-dung'].UPDATE, this.form).then(rs => {
          this.tempData = rs
          this.form = rs
          this.isHiddenInput = !this.isHiddenInput
          showAlert(this, SUCCESS, 'Cập nhật thành công!')
        }).catch(err => {
          errAlert(this, err)
        }).finally(() => {
          this.loading = false
        })
      })
    },

    onClearCitizenId(val) {
      if (!val) {
        this.$refs.form.clearValidate(['citizenDate', 'citizenIssuedBy'])
      }
    },

    preventChar,

    resetForm(formName) {
      this.isHiddenInput = !this.isHiddenInput
      this.$refs[formName].clearValidate()
      this.form = this.tempData
    }
  }
}
</script>
