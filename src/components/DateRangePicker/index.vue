<template>
  <el-date-picker
      v-model="val"
      :format="format || DATE.DD_MM_YYYY2"
      popper-class="fixSizeWhenScale"
      start-placeholder="Từ ngày"
      end-placeholder="Đến ngày"
      range-separator="Đến"
      style="width: 100%"
      :clearable="clearable"
      :disabled="disabled"
      :unlink-panels="true"
      :picker-options="pickerOptions"
      type="daterange"
      @input="onInput"
      @change="onChange"
      @blur="onBlur"
      @focus="onFocus"
  />
</template>

<script>
import {DATE} from 'ff24-js/src/utils/Constant';

export default {
  name: 'DateRangePicker',
  data() {
    return {
      DATE
    }
  },
  props: {
    value: {
      type: [Array],
      default: () => {
        return []
      }
    },
    format: String,
    clearable: {
      type: Boolean,
      default: false
    },
    disabled: Boolean,
    pickerOptions: Object
  },
  computed: {
    val: {
      get() {
        return this.value
      },
      set(v) {
        this.$emit('input', v)
      }
    }
  },
  methods: {
    onChange(val) {
      this.$emit('change', val)
    },
    onBlur(val) {
      this.$emit('blur', val)
    },
    onFocus(val) {
      this.$emit('focus', val)
    },
    onInput(val) {
      if (!val) {
        this.$emit('input', [])
      }
    }
  }
}
</script>

<style>
@media only screen and (max-width: 1366px) {
  .fixSizeWhenScale {
    -webkit-transform: scale(0.8);
  }
}
</style>
