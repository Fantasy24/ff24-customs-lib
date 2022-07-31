<template>
  <div
      :class="{'is-disabled':disabled,'el-input-group el-input-group--append':!!$slots.append}"
      class="money el-input"
  >
    <div class="el-input--mini el-input--suffix">
      <input
          ref="ipt"
          v-model="val"
          :disabled="disabled"
          :maxlength="maxlength"
          :placeholder="placeholder"
          :validateevent="true"
          autocomplete="off"
          class="el-input__inner"
          style="line-height: 28px !important;"
          type="text"
          @blur="blur"
          @focus="focus"
          @keypress="onKeypress"
          @paste.prevent
      >
      <span v-show="showClose" class="el-input__suffix">
        <span class="el-input__suffix-inner">
          <em
              class="el-input__icon el-icon-circle-close el-input__clear"
              @click="clear"
          />
        </span>
      </span>
    </div>
    <div
        v-if="!!$slots.append"
        class="el-input-group__append"
    >
      <slot name="append"/>
    </div>
  </div>
</template>
<script>
function numToMoney(v, n) {
  const fixed = this.fixed || 2
  let r
  const ipt = this.$refs['ipt']
  if (v === '' || v === undefined) {
    return v
  } else {
    const [a, b = ''] = (v + '').split(/(?=\.)/)
    const x = a.length % 3
    r = a.substr(0, x) +
        a.substr(x).replace(/(\d{3})/g, ',$1').substr(+!x)
    if (n) {
      r += (+(0 + b)).toFixed(n).substr(1)
    } else if (b) r += b.substr(0, fixed + 1)
  }
  if (ipt) {
    let st = -1
    if (document.activeElement === ipt) {
      const val = ipt.value + ''
      if (val === '' || /^\.0*$/g.test(val)) {
        this.$emit('input', '')
        this.$emit('change', '')
        return ''
      }
      const point = val.indexOf('.')
      const l0 = val.length
      const l1 = r.length
      const fix = l1 - l0
      const s0 = ipt.selectionStart
      const s1 = ipt.selectionEnd
      if (point !== -1 && s0 > point) {
        st = s0 - fix + (
            fix > 0
                ? s0 === s1
                    ? s0 - point === fix && fix === 1
                        ? 0 : 1
                    : 2
                : -1
        )
      } else {
        if (val.length === 1 && s0 === s1 && fix === fixed + 1) {
          st = 1
        } else {
          st = s0 + fix
        }
      }
    }
    ipt.value = r
    if (st !== -1) ipt.setSelectionRange(st, st)
  }
  return Number(r) <= 0 ? '' : r
}

import {moneyPrevent, preventChar} from 'ff24-js/src/utils/ECustomsUtils'

export default {
  props: {
    value: {
      required: false,
      'default': ''
    },
    placeholder: {
      default: ''
    },
    disabled: {
      type: Boolean,
      required: false,
      'default': false
    },
    fixed: {
      type: Number,
      required: false,
      default: 0
    },
    max: {
      type: Number,
      required: false
    },
    maxlength: {
      type: [String, Number],
      required: false
    },
    min: {
      type: Number,
      required: false
    },
    clearable: {
      type: Boolean,
      required: false,
      'default': false
    }
  },
  data() {
    return {
      showClose: false
    }
  },
  computed: {
    val: {
      get() {
        this.showClose = this.clearable && this.value !== '' && !this.disabled
        return numToMoney.call(this, this.value, this.fixed)
      },
      set(v) {
        const {fixed = 2, value, max = Number.MAX_SAFE_INTEGER} = this
        const fix1 = v.replace(/[^0-9,.]/g, '')
        const fix2 = v.replace(/,/g, '')
        const fix3 = parseFloat(Math.min(parseFloat(fix2), max).toFixed(fixed))
        const nan = isNaN(fix3)
        if (fix1 !== v || nan || value === fix3) {
          numToMoney.call(this, value, fixed)
        } else if (!nan) {
          this.$emit('input', fix3)
          this.$emit('change', fix3)
        }
      }
    }
  },
  mounted() {
    const {$refs: {ipt}, min = Number.MIN_SAFE_INTEGER} = this
    ipt.addEventListener('blur', ({target: {value}}) => {
      this.val = Math.max(value, min) + ''
    })
  },
  methods: {
    onKeypress(e) {
      this.moneyPrevent(e)
      let value = e.target.value
      const valueSplit = value.split('.')
      const maxLengthBegin = this.max ? String(this.max).length : 20

      const cursorIndex = this.$refs.ipt.selectionStart
      const lastDotIndex = value.lastIndexOf('.')

      if (valueSplit && valueSplit.length > 0 && this.fixed > 0) {
        const beginDot = valueSplit[0].replaceAll(',', '')
        const afterDot = valueSplit[1] || String(this.fixed).length.toString()
        const checkAfterDot = Number(afterDot.slice(-1)) > 1
        if ((beginDot.length <= maxLengthBegin && checkAfterDot && cursorIndex > lastDotIndex)) {
          e.preventDefault()
        } else if ((beginDot.length >= maxLengthBegin && e.charCode !== 46 /* charCode 46: dấu chấm */) || checkAfterDot) {
          if ((cursorIndex <= lastDotIndex || checkAfterDot) && beginDot.length >= maxLengthBegin) {
            e.preventDefault()
          }
        }
      } else {
        value = value.replaceAll(',', '')
        if (value.length >= maxLengthBegin) {
          e.preventDefault()
        }
      }
    },

    clear() {
      const {ipt} = this.$refs
      ipt.focus()
      ipt.value = ''
      this.$emit('input', '')
      this.$emit('change', '')
    },

    focus() {
      this.$emit('focus')
    },

    blur(e) {
      if (!e.target.value) {
        this.$refs.ipt.value = ''
        this.$emit('input', '')
        this.$emit('change', '')
      }
      this.$emit('blur', e)
    },
    moneyPrevent,
    preventChar
  }
}
</script>
