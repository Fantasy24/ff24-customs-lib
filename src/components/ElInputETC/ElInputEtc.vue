<template>
  <el-input
      :id="id"
      :ref="refInput"
      v-model="val"
      :clearable="clearable"
      :disabled="disabled"
      :maxlength="maxlength"
      :minlength="minlength"
      :placeholder="placeholder"
      :readonly="readonly"
      :rows="rows"
      :show-password="showPassword"
      :show-word-limit="showWordLimit"
      :size="size"
      :tabindex="tabindex"
      :type="type"
      @blur="handleBlur"
      @change="handleChange"
      @compositionend="handleCompositionEnd"
      @compositionstart="handleCompositionStart"
      @compositionupdate="handleCompositionUpdate"
      @focus="handleFocus"
      @input="handleInput"
  />
</template>

<script>
export default {
  name: 'ElInputEtc',
  props: {
    id: {
      type: String,
      default: () => {
        return 'idInput' + Math.floor(Math.random() * 1000)
      }
    },
    refInput: {
      type: String,
      default: ''
    },
    value: {
      type: [String, Number],
      default: ''
    },
    vModel: {
      type: [String, Number],
      default: ''
    },
    clearable: {
      type: Boolean,
      default: true
    },
    showWordLimit: {
      type: Boolean,
      default: true
    },
    type: {
      type: String,
      default: 'text'
    },
    showPassword: Boolean,
    size: String,
    disabled: Boolean,
    tabindex: String,
    readonly: Boolean,
    maxlength: [String, Number],
    minlength: [String, Number],
    placeholder: String,
    rows: Number
  },
  computed: {
    val: {
      get() {
        let strValue = ''
        if (!this.vModel) {
          strValue = this.value ? String(this.value) : ''
        } else {
          strValue = this.vModel ? String(this.vModel) : ''
        }
        return strValue.trimLeft()
      },
      set(value) {
        this.$emit('update:vModel', value)
        this.$emit('input', value)
      }
    }
  },
  methods: {
    handleCompositionStart(event) {
      this.$emit('compositionstart', event)
    },
    handleCompositionUpdate(event) {
      this.$emit('compositionupdate', event)
    },
    handleCompositionEnd(event) {
      this.$emit('compositionend', event)
    },
    handleInput(event) {
      this.$emit('input', event)
    },
    handleFocus(event) {
      this.$emit('focus', event)
    },
    handleBlur(event) {
      this.val = this.val.trim()
      this.$emit('blur', event)
    },
    handleChange(event) {
      this.$emit('input', event)
      this.$emit('change', event)
    }
  }
}
</script>

<style scoped>

</style>
