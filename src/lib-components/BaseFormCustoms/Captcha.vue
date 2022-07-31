<template>
  <div>
    <el-row :gutter="rowGutter">
      <el-col :span="colSpanInput">
        <el-form-item :prop="propForm" :required="required" :rules="rules">
              <span class="svg-container">
                <svg-icon icon-class="captcha"/>
              </span>
          <el-input
              :id="id"
              :ref="ref"
              v-model="valueSync"
              :disabled="disabled"
              :placeholder="$t('login.captcha')"
              maxlength="8"
              tabindex="9"
              @keyup.enter.native="onKeyupCaptcha"
          />
        </el-form-item>
      </el-col>

      <el-col :span="colSpanImg">
        <img
            id="captchaImg"
            :src="`data:image/png;base64,${codeCaptcha}`"
            :title="$t('login.codeCaptcha')"
            alt="ct"
            style="width: 100%;height: 50px;font-size: 11px"
            @click="refreshCaptcha"
        >
      </el-col>
    </el-row>
  </div>
</template>

<script>
import apiFactory from 'ff24-js/src/api/apiFactory';
import {errAlert} from 'ff24-js/src/utils/AlertMessage';
import ConstantAPI from 'ff24-js/src/utils/ConstantAPI';

const validateCaptcha = (rule, value, callback) => {
  if (!value) {
    callback(new Error(this.$t('login.validCaptcha')));
  } else if (value.length > 8) {
    callback(new Error(this.$t('login.maxlengthCc')));
  } else {
    callback();
  }
};

export default {
  props: {
    value: {
      type: String,
      required: true,
      default: ''
    },
    form: {
      type: Object,
      required: true,
      default: () => {}
    },
    id: {
      type: String,
      default: 'captchaKey'
    },
    ref: {
      type: String,
      default: 'captchaKey'
    },
    required: {
      type: Boolean,
      default: true
    },
    disabled: {
      type: Boolean,
      default: false
    },
    rules: {
      type: Array,
      default: () => {
        return {captchaKey: [{required: true, validator: validateCaptcha, trigger: 'change'}]}
      }
    },
    propForm: {
      type: String,
      required: true,
      default: 'captchaKey'
    },
    formRef: {
      type: String,
      required: true,
      default: ''
    },
    onKeyupCaptcha: {
      type: Function,
      default: () => {}
    },
    colSpanInput: {
      type: Number,
      default: 12
    },
    colSpanImg: {
      type: Number,
      default: 12
    },
    rowGutter: {
      type: Number,
      default: 10
    }
  },
  data() {
    return {
      codeCaptcha: '',
      loading: false
    };
  },
  computed: {
    valueSync: {
      get() {
        return this.value;
      },
      set(val) {
        this.$emit('update:value', val);
      }
    },
    formSync: {
      get() {
        return this.form;
      },
      set(form) {
        this.$emit('update:form', form);
      }
    }
  },
  methods: {
    refreshCaptcha() {
      apiFactory.callAPI(ConstantAPI.ACCOUNT_CHANGE.GET_CAPTCHA).then(rs => {
        this.codeCaptcha = rs['captchaImage'];
        this.formSync.captchaSecret = rs.captchaSecret;
      }).catch(err => {
        errAlert(this, err);
      });
    }
  }
};
</script>

<style scoped>

</style>

