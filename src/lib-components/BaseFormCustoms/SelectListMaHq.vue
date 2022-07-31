<template>
  <div>
    <el-form-item
        :label="label"
        :label-width="labelWidth"
        :prop="propForm"
        :required="isRequire || required"
        :rules="isRequire || required ? rules : null"
    >
      <el-select
          :id="propForm"
          v-model="selectMaHq"
          :clearable="clearable"
          :disabled="disabled || isDisable"
          :filterable="filterable"
          :loading="loading"
          :placeholder="placeholder || label"
          :popper-class="popperClass"
          style="width: 100%"
          @change="onChange"
          @clear="onClear"
      >
        <el-option v-if="isShowOptionAll" :label="labelOption" :value="'' || `_${$store.getters.org}`"/>
        <el-option
            v-for="item in listMahq"
            :key="item.code"
            :label="`${item.code} - ${item.name}`"
            :value="item.code"
        />
      </el-select>
    </el-form-item>
  </div>
</template>

<script>
import {LIST_CUSTOMS} from 'ff24-js/src/utils/ECustomsUtils'
import {SEARCH_RULES} from 'ff24-js/src/utils/KeySearchListObj'

export default {
  name: 'SelectMaHq',
  props: {
    isFirst: Boolean,
    isGetAll: {
      type: Boolean,
      default: false
    },
    except: Array,
    popperClass: {
      type: String,
      default: ''
    },
    isShowOptionAll: {
      type: Boolean,
      default: true
    },
    onlyCurrentUser: {
      type: Boolean,
      default: false
    },
    sCusid: {
      type: String,
      default: ''
    },
    vModel: {
      type: String,
      default: ''
    },
    label: {
      type: String,
      default: 'Mã HQ'
    },
    placeholder: String,
    labelOption: {
      type: String,
      default: '--Tất cả--'
    },
    isRequire: Boolean,
    required: {
      type: Boolean,
      default: true
    },
    clearable: {
      type: Boolean,
      default: true
    },
    filterable: {
      type: Boolean,
      default: true
    },
    disabled: {
      type: Boolean,
      default: false
    },
    isDisable: Boolean,
    rules: {
      type: Array,
      default: () => {
        return SEARCH_RULES.orgCode
      }
    },
    propForm: {
      type: String,
      default: ''
    },
    labelWidth: {
      type: String,
      default: ''
    },
    /* Chỉ hiện chi cục*/
    onlyCc: Boolean
  },
  data() {
    return {
      listMahq: [],
      loading: false
    }
  },
  computed: {
    selectMaHq: {
      get() {
        return this.sCusid || this.vModel
      },
      set(value) {
        this.$emit('update:sCusid', value)
        this.$emit('update:vModel', value)
      }
    }
  },
  mounted() {
    this.listMahq = this.getListHQByOrgCode()
    if (this.isFirst) {
      document.getElementById(this.propForm).focus()
    }
  },
  methods: {
    getListHQByOrgCode() {
      this.loading = true
      const maHq = this.$store.getters.org.trim()
      const orgList = JSON.parse(localStorage.getItem(LIST_CUSTOMS))

      if (this.onlyCc) {
        this.listMahq = orgList.filter(item => !item.code.includes('ZZ'))
        this.loading = false
        return this.listMahq
      }

      if (this.onlyCurrentUser) {
        this.listMahq = orgList.filter(item => item.code.includes(maHq))
      } else if (this.isGetAll || maHq === '00ZZ') {
        this.listMahq = orgList
      } else if (maHq.includes('ZZ')) {
        this.listMahq = orgList.filter(item => item.code.includes(maHq.substr(0, 2)))
      } else {
        this.listMahq = orgList.filter(item => item.code === maHq)
      }

      if (this.except) {
        this.listMahq = this.listMahq.filter(e => !this.except.includes(e.code))
      }
      this.loading = false
      return this.listMahq
    },
    onChange(valueOrg) {
      this.$emit('onChange', valueOrg)
      this.$emit('change', valueOrg)
    },

    onClear() {
      this.$emit('update:sCusid', '')
      this.$emit('clear', '')
    }
  }

}
</script>

<style scoped>

</style>
