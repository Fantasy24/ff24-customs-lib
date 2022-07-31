<template>
  <div>
    <el-form-item :label="label" :prop="propForm" :required="isRequire" :rules="rules">
      <el-select
          v-model="selectPosCode"
          :disabled="disabled"
          clearable
          filterable
          placeholder="Mã chức vụ"
          style="width: 100%">
        <el-option
            v-for="item in listPosition"
            :key="item.code"
            :label="`${item.code} - ${item.name}`"
            :value="item.code"
        />
      </el-select>
    </el-form-item>
  </div>
</template>

<script>
import {LIST_POSITIONS} from 'ff24-js/src/utils/ECustomsUtils';

export default {
  name: 'SelectPosCode',
  props: {
    posCode: {
      type: String,
      default: ''
    },
    label: {
      type: String,
      default: 'Mã chức vụ'
    },
    isRequire: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    rules: {
      type: Array,
      default: () => []
    },
    propForm: {
      type: String,
      default: ''
    },
    store: {
      type: Object,
      required: true,
      default: () => {
      }
    }
  },
  data() {
    return {
      listPosition: JSON.parse(localStorage.getItem(LIST_POSITIONS))
    };
  },
  computed: {
    selectPosCode: {
      get() {
        return this.posCode;
      },
      set(val) {
        this.$emit('update:posCode', val);
      }
    }
  }
};
</script>

<style scoped>

</style>
