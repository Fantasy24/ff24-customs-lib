<template>
  <el-dialog
      :append-to-body="appendToBody"
      :before-close="beforeClose"
      :center="center"
      :close-on-click-modal="closeOnClickModal"
      :close-on-press-escape="closeOnPressEscape"
      :custom-class="customClass"
      :destroy-on-close="destroyOnClose"
      :fullscreen="fullscreen"
      :lock-scroll="lockScroll"
      :modal="modal"
      :modal-append-to-body="modalAppendToBody"
      :show-close="showClose"
      :title="title"
      :top="top"
      :visible.sync="syncVisible"
      :width="width"
      @close="onClose"
      @closed="onClosed"
      @open="onOpen"
      @opened="onOpened"
  >
    <span slot="title">
      {{ title }}
      <el-tooltip :content="fullscreen ? 'Mặc định' : 'Toàn màn hình'" :open-delay="100" effect="light">
        <el-button
            size="medium"
            style="float: right;margin-right: 25px;margin-top: -10px;"
            type="text"
            @click="fullscreen = !fullscreen"
        >
          <svg-icon :icon-class="fullscreen?'exit-fullscreen':'fullscreen'"/>
        </el-button>
      </el-tooltip>
    </span>
    <slot/>
    <span slot="footer" class="dialog-footer">
      <slot name="footer"/>
      <el-button icon="el-icon-close" @click="onClose">{{ $t('baseLabel.btnCancel') }}</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  props: {
    visible: Boolean,
    closeOnClickModal: Boolean,
    appendToBody: Boolean,
    center: Boolean,
    customClass: String,
    destroyOnClose: Boolean,
    showClose: {
      type: Boolean,
      default: true
    },
    lockScroll: {
      type: Boolean,
      default: true
    },
    modal: {
      type: Boolean,
      default: true
    },
    modalAppendToBody: {
      type: Boolean,
      default: true
    },
    closeOnPressEscape: {
      type: Boolean,
      default: true
    },
    width: {
      type: String,
      default: '50%'
    },
    title: String,
    top: String,
    beforeClose: Function
  },
  data() {
    return {
      fullscreen: false
    }
  },
  computed: {
    syncVisible: {
      get() {
        return this.visible
      },
      set(val) {
        this.$emit('update:visible', val)
      }
    }
  },
  methods: {
    onClose($event) {
      this.syncVisible = false
      this.$emit('close', $event)
    },
    onClosed($event) {
      this.$emit('closed', $event)
    },
    onOpen($event) {
      this.$emit('open', $event)
    },
    onOpened($event) {
      this.$emit('opened', $event)
    }
  }
}
</script>

<style scoped>

</style>
