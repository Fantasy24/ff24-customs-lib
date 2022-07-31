<template>
  <div>
    <el-form-item :label="label" :prop="propForm" :required="isRequire" :rules="rules">
      <el-select
          v-model="selectDepCode"
          :disabled="disabled"
          clearable
          filterable
          placeholder="Mã phòng ban"
          style="width: 100%">
        <el-option
            v-for="item in listDepCode"
            :key="item.code"
            :label="`${item.code} - ${item.name}`"
            :value="item.code"
        />
      </el-select>
    </el-form-item>
  </div>
</template>

<script>
import {LIST_DEPARTMENT} from 'ff24-js/src/utils/ECustomsUtils';

export default {
  name: 'SelectDepCode',
  props: {
    depCode: {
      type: String,
      default: ''
    },
    label: {
      type: String,
      default: 'Mã phòng ban'
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
      listDepCode: JSON.parse(localStorage.getItem(LIST_DEPARTMENT))
    };
  },
  computed: {
    selectDepCode: {
      get() {
        return this.depCode;
      },
      set(val) {
        this.$emit('update:depCode', val);
      }
    }
  }
};
</script>

<style scoped>

</style>
