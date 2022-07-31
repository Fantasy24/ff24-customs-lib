<template>
  <el-dropdown
      v-if="isExport && total > 0"
      id="dropdown"
      v-loading="loading"
      class="drowdown"
      split-button
      trigger="click"
      @click="handleExportDefault"
      @command="handleCommand"
  >
    <svg-icon id="mainIcon" icon-class="excel"></svg-icon>
    <strong class="space"> Excel</strong>
    <el-dropdown-menu slot="dropdown">
      <el-dropdown-item
          v-for="(file, index) in fileTypes"
          :key="index"
          :command="{method: 'handleExport', format: file.type}"
          :divided="index !== 0"
      >
        <svg-icon :icon-class="file.icon"/>
        <strong class="space">{{ `[.${file.type}]` }}</strong>
      </el-dropdown-item>
    </el-dropdown-menu>
  </el-dropdown>
</template>

<script>
import {deepClone} from 'ff24-js/src/utils'
import apiFactory, {isValidDateTime} from 'ff24-js/src/api/apiFactory'
import ConstantAPI from 'ff24-js/src/utils/ConstantAPI'
import {errAlert, showAlert, SUCCESS} from 'ff24-js/src/utils/AlertMessage'
import moment from 'moment'
import {EXPORT_FILE_TYPES} from 'ff24-js/src/utils/Constant'

export default {
  name: 'DropdownExport',
  data() {
    return {
      EXPORT_FILE_TYPES,
      loading: false,
      notChangeParam: false,
      formExport: deepClone(FORM_REQUEST)
    }
  },
  props: {
    isExport: Boolean,
    total: Number,
    apiFetch: Object,
    paramsFetch: Object,
    exportFileType: [Array, Object],
    menuCode: String
  },
  watch: {
    paramsFetch: {
      handler() {
        this.notChangeParam = false
      },
      deep: true
    }
  },
  computed: {
    fileTypes() {
      if (!this.exportFileType) {
        return EXPORT_FILE_TYPES
      } else if (typeof this.exportFileType === 'object' && !this.exportFileType.length) {
        return [this.exportFileType]
      } else {
        return this.exportFileType
      }
    }
  },
  methods: {
    handleExportDefault($event) {
      this.loading = true
      this.$emit('click', $event)
      setTimeout(() => {
        this.loading = false
      }, 200)
    },
    handleCommand(command) {
      this[command.method](command.format)
    },
    handleExport(format) {
      this.loading = true
      this.formExport.fileFormat = format
      this.formExport.exportApiUrl = this.apiFetch.url

      /* format lại kiểu ngày tháng trc khi parse String object*/
      for (const [key, value] of Object.entries(this.paramsFetch)) {
        this.paramsFetch[key] = isValidDateTime(value) ? moment(value).format('YYYY/MM/DD HH:mm:ss') : value
      }
      delete this.paramsFetch.fromToDate
      delete this.paramsFetch.fromToDate2

      this.formExport.exportApiData = JSON.stringify(this.paramsFetch)
      this.formExport.menuCode = this.menuCode
      this.formExport.orgCode = this.$store.getters.org
      apiFactory.callAPI(ConstantAPI.EXPORT.EXPORT_REQUEST, this.formExport).then(() => {
        showAlert(this, SUCCESS, `Đã tiếp nhận yêu cầu kết xuất  [${format.toUpperCase()}], chờ xử lý`, 6000)
        this.notChangeParam = true
      }).catch(err => {
        console.log('handleExport', 'src/components/ElTableETC/DropdownExport.vue')
        errAlert(this, err)
        this.notChangeParam = false
      }).finally(() => {
        this.loading = false
      })
    }
  }
}

const FORM_REQUEST = {
  appCode: process.env.VUE_APP_APP_CODE,
  exportApiData: '',
  exportApiUrl: '',
  fileFormat: '',
  menuCode: '',
  orgCode: ''
}

</script>

<style scoped>
#dropdown >>> .el-loading-spinner .circular {
  height: 25px !important;
}

#dropdown >>> .el-loading-spinner {
  margin-top: -11px;
}

#mainIcon {
  vertical-align: -0.16em !important;
}

.space {
  margin-left: 10px;
}

.el-dropdown-menu__item--divided {
  background-color: #efefef !important;
}

.el-dropdown-menu__item {
  font-weight: 550 !important;
  color: #070946 !important;
}

.el-dropdown-menu__item.is-disabled {
  color: #bbb !important;
}

.el-dropdown-menu__item:hover {
  font-weight: bold !important;
  color: #363636 !important;
  background-color: ghostwhite !important;
}

.el-dropdown-menu {
  background-color: #efefef !important;
  border: 1px solid #efefef !important;
}
</style>
