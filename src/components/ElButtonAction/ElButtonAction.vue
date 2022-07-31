<template>
  <el-tooltip :content="describe" :open-delay="openDelay" placement="top" effect="light">
    <el-button
      :loading="loading"
      :disabled="buttonDisabled || loading"
      :autofocus="autofocus"
      circle
      plain
      :type="type"
      size="mini"
      :icon="elIcon"
      @click="handleClick"
    >
      <svg-icon v-if="svgIcon" :icon-class="svgIcon" />
    </el-button>
  </el-tooltip>
</template>

<script>
export default {
  name: 'ElButtonAction',
  inject: {
    elForm: {
      default: ''
    },
    elFormItem: {
      default: ''
    }
  },

  props: {
    type: {
      type: String,
      default: 'primary'
    },
    openDelay: {
      type: Number,
      default: 400
    },
    svgIcon: {
      type: String,
      default: ''
    },
    elIcon: {
      type: String,
      default: ''
    },
    describe: {
      type: String,
      required: true,
      default: 'Tên action'
    },
    loading: Boolean,
    disabled: Boolean,
    autofocus: Boolean
  },

  computed: {
    buttonDisabled() {
      return this.disabled || (this.elForm || {}).disabled
    }
  },

  methods: {
    handleClick(evt) {
      this.$emit('click', evt)
    }
  }
}
</script>

<style scoped>

</style>
