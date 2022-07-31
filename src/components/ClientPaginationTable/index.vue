<template>
  <div class="client-table">
    <el-table
        ref="elTable"
        v-loading="syncLoadData"
        :border="border"
        :cell-class-name="cellClassName"
        :cell-style="cellStyle"
        :current-row-key="currentRowKey"
        :data="syncData"
        :default-expand-all="defaultExpandAll"
        :default-sort="defaultSort"
        :empty-text="emptyText"
        :expand-row-keys="expandRowKeys"
        :fit="fit"
        :header-cell-class-name="headerCellClassName"
        :header-cell-style="headerCellStyle"
        :header-row-class-name="headerRowClassName"
        :header-row-style="headerRowStyle"
        :height="height"
        :highlight-current-row="highlightCurrentRow"
        :indent="indent"
        :lazy="lazy"
        :load="load"
        :max-height="maxHeight"
        :row-class-name="rowClassName"
        :row-key="rowKey"
        :row-style="rowStyle"
        :select-on-indeterminate="selectOnIndeterminate"
        :show-header="showHeader"
        :show-summary="showSummary"
        :size="tableSize"
        :span-method="calSpan"
        :stripe="stripe"
        :sum-text="sumText"
        :summary-method="summaryMethod"
        :tooltip-effect="tooltipEffect"
        :tree-props="treeProps"
        :width="width"
        style="width : 100%"
        @select="select"
        @select-all="selectAll"
        @selection-change="selectionChange"
        @cell-mouse-enter="cellMouseEnter"
        @cell-mouse-leave="cellMouseLeave"
        @cell-click="cellClick"
        @row-click="rowClick"
        @row-contextmenu="rowContextmenu"
        @row-dblclick="rowDbclick"
        @header-click="headerClick"
        @header-contextmenu="headerContextmenu"
        @sort-change="sortChange"
        @filter-change="filterChange"
        @current-change="currentChange"
        @header-dragend="headerDragend"
        @expand-change="expandChange"
    >
      <el-table-column
          v-if="isSelectable"
          align="center"
          fixed
          type="selection"
          width="45"
      />

      <slot name="expand"/>

      <el-table-column
          v-if="showStt"
          align="center"
          label="STT"
          width="55"
      >
        <template slot-scope="{row, $index}">
          <span>{{ countStt(row, $index) }}</span>
        </template>
      </el-table-column>

      <slot name="columnsTop"/>

      <el-table-column
          v-for="col in renderTableColumns"
          :key="col.prop"
          :align="col.align"
          :class-name="col.className"
          :column-key="col.columnKey"
          :filter-method="col.filterMethod"
          :filter-multiple="col.filterMultiple"
          :filter-placement="col.filterPlacement"
          :filtered-value="col.filteredValue"
          :filters="col.filters"
          :fixed="col.fixed"
          :formatter="typeof col.formatter === 'function' ? col.formatter : null"
          :header-align="col.headerAlign"
          :index="col.index"
          :label="col.label"
          :label-class-name="col.labelClassName"
          :min-width="col.minWidth || col.width"
          :prop="col.prop"
          :property="col.property"
          :render-header="col.renderHeader"
          :reserve-selection="col.reserveSelection"
          :resizable="col.resizable"
          :selectable="col.selectable"
          :show-overflow-tooltip="col.showOverflowTooltip = true"
          :show-tooltip-when-overflow="col.showTooltipWhenOverflow = true"
          :sort-by="col.sortBy"
          :sort-method="col.sortMethod"
          :sort-orders="col.sortOrder"
          :sortable="col.sortable"
          :type="col.type"
      >
        <template v-if="typeof col.formatter !== 'function'" #default="{row}">
          <div
              :is="col.formatter"
              v-if="col.formatter"
              :column="col"
              :row="row"
          />

          <template v-else>
            <read-more
                v-if="row[col.prop] && String(row[col.prop]).includes('\n')"
                id="readMore"
                :max-chars="checkEscapeSequence(row[col.prop], col)"
                :text="row[col.prop]"
                less-str="Ẩn"
                more-str="Xem thêm"
            />
            <span v-else> {{ row[col.prop] }} </span>
          </template>
        </template>
      </el-table-column>

      <slot name="columns"/>

    </el-table>

    <slot name="content"/>

    <el-card
        v-if="isShowUtil && (isExport || isShowPdf || isShowPrint || isShowHideColumn)"
        id="formUtil"
        class="utilStyle"
        shadow="never"
    >
      <div>
        <span v-if="!dropdownUtil">
          <el-button
              v-if="isExport && syncTotal > 0"
              :loading="loadExcel"
              circle
              size="mini"
              title="Xuất Excel"
              type="text"
              @click="exportExcel"
          >
            <svg-icon icon-class="excel"/>
          </el-button>

          <el-button
              v-if="isShowPdf && syncTotal > 0"
              :loading="loadPdf"
              circle
              size="mini"
              title="Xuất PDF"
              type="text"
              @click="onExportPDF"
          >
            <svg-icon icon-class="pdf"/>
          </el-button>

          <print
              v-if="isShowPrint && syncTotal > 0"
              :columns="renderTableColumns"
              :date-from="dateFrom"
              :date-to="dateTo"
              :export-title="reportTitle"
              :remote-method="remoteMethod || onGetDataPrint"
          />
        </span>
        <span v-else>
          <dropdown-export
              :api-fetch="apiFetch"
              :export-file-type="exportFileType"
              :is-export="isExport"
              :menu-code="menuCode"
              :params-fetch="paramsFetch"
              :total="syncTotal"
              @click="exportExcel"
          />
        </span>

        <el-divider v-if="isExport && syncTotal > 0" direction="vertical"/>

        <el-popover
            v-if="isShowHideColumn"
            placement="bottom"
            trigger="click"
            width="auto"
        >
          <div class="checkedColumns">
            <div style="text-align: center !important;margin-bottom: 5px">
              <el-checkbox
                  v-model="checkAll"
                  :indeterminate="isIndeterminate"
                  @change="handleCheckedAll"
              >
                Tất cả
              </el-checkbox>
            </div>
            <el-checkbox-group
                v-model="checkedTableColumns"
                :min="minChecked"
                @change="handleCheckedColumnsChange"
            >
              <el-row
                  v-for="col in columns"
                  :key="col.prop"
                  :gutter="10"
                  style="margin-bottom: 4px"
              >
                <el-checkbox :disabled="col.disabled" :label="col.prop">{{ col.label }}</el-checkbox>
              </el-row>
            </el-checkbox-group>
          </div>
          <el-button
              slot="reference"
              icon="el-icon-setting"
              plain
              size="mini"
          >
            Chọn chỉ tiêu
          </el-button>
        </el-popover>
      </div>
    </el-card>

    <pagination
        v-if="isPagination"
        v-show="syncTotal > 0"
        :limit.sync="size"
        :page.sync="page"
        :total="syncTotal"
        @pagination="pagination"
    />
  </div>
</template>

<script>
/* eslint-disable vue/require-default-prop */
import {PAGINATION_PARAM} from 'ff24-js/src/utils/Constant'
import pdfMaker from 'pdfmake/build/pdfmake'
import pdfFonts from 'pdfmake/build/vfs_fonts'
import {checkDate, formatDate, formatFullDateTime_VN, replaceEscapeChar} from 'ff24-js/src/filters'
import moment from 'moment'
import apiFactory from 'ff24-js/src/api/apiFactory'
import {errAlert} from 'ff24-js/src/utils/AlertMessage'
import {deepClone} from 'ff24-js/src/utils'
import Print from 'etc-customs-lib/src/components/Print/Print'
import Pagination from 'etc-customs-lib/src/components/Pagination/index.vue'
import {export_json_to_excel} from 'ff24-js/src/vendor/Export2Excel'
import ReadMore from 'vue-read-more/components/ReadMoreComponent.vue'
import ReadMoreDirective from 'vue-read-more/directives/ReadMoreDirective.js'
import DropdownExport from 'etc-customs-lib/src/components/ElTableETC/DropdownExport.vue'
import bigDecimal from 'js-big-decimal'

export default {
  name: 'ClientPaginationTable',
  components: {Print, Pagination, DropdownExport, ReadMore},
  directives: {
    readMore: ReadMoreDirective
  },
  props: {
    tableSize: {
      type: String,
      default: 'mini'
    },
    emptyText: {
      type: String,
      default: 'Không có dữ liệu'
    },
    loading: {
      type: Boolean,
      default: false
    },
    columns: {
      type: Array,
      default: () => []
    },
    isPagination: {
      type: Boolean,
      default: true
    },
    isSelectable: {
      type: Boolean,
      default: false
    },
    showStt: {
      type: Boolean,
      default: true
    },
    /* Table*/
    width: [String, Number],
    height: [String, Number],
    maxHeight: [String, Number],
    fit: {
      type: Boolean,
      default: true
    },
    stripe: Boolean,
    border: {
      type: Boolean,
      default: true
    },
    rowKey: [String, Function],
    showHeader: {
      type: Boolean,
      default: true
    },
    showSummary: Boolean,
    sumText: String,
    summaryMethod: Function,
    rowClassName: [String, Function],
    rowStyle: [Object, Function],
    cellClassName: [String, Function],
    cellStyle: [Object, Function],
    headerRowClassName: [String, Function],
    headerRowStyle: [Object, Function],
    headerCellClassName: [String, Function],
    headerCellStyle: [Object, Function],
    highlightCurrentRow: {
      type: Boolean,
      default: false
    },
    currentRowKey: [String, Number],
    expandRowKeys: Array,
    defaultExpandAll: Boolean,
    defaultSort: Object,
    tooltipEffect: String,
    selectOnIndeterminate: {
      type: Boolean,
      default: true
    },
    indent: {
      type: Number,
      default: 16
    },
    treeProps: {
      type: Object,
      default() {
        return {
          hasChildren: 'hasChildren',
          children: 'children'
        }
      }
    },
    lazy: Boolean,
    load: Function,
    isSttColumn: {
      type: Boolean,
      default: true
    },
    /* Table*/

    /* Export*/
    isShowHideColumn: {
      type: Boolean,
      default: true
    },
    isShowUtil: {
      type: Boolean,
      default: true
    },
    exportFileType: [Array, Object],
    dropdownUtil: {
      type: Boolean,
      default: false
    },
    isExport: {
      type: Boolean,
      default: false
    },
    isExportTemplate: {
      type: Boolean,
      default: false
    },
    isShowPdf: {
      type: Boolean,
      default: false
    },
    isShowPrint: {
      type: Boolean,
      default: false
    },
    constantApiPdf: {
      type: Object,
      default() {
        return {
          url: '',
          method: 'GET',
          params: {}
        }
      }
    },
    constantApiExcel: {
      type: Object,
      default() {
        return {
          url: '',
          method: 'POST',
          params: {}
        }
      }
    },
    menuCode: String,
    rsProp: String,
    allData: {
      type: Array,
      default: () => []
    },
    filterEmptyRowData: Boolean,
    apiFetch: {
      type: Object,
      default: null
    },
    paramsFetch: {
      type: Object,
      default: null
    },
    fileName: {
      type: String,
      default: ''
    },
    reportTitle: {
      type: String,
      default: ''
    },
    remoteMethod: Function,
    dateFrom: {
      type: Date,
      default() {
        return new Date()
      }
    },
    dateTo: {
      type: Date,
      default() {
        return new Date()
      }
    },
    sumTitle: {
      type: String,
      default: () => 'Tổng'
    },
    pageSizePdf: {
      type: String,
      default: 'A4'
    }, /* Size khổ giấy pdf*/
    arrsColIgnoreExport: Array /* list tên cột không export*/
    /* Export*/
  },
  data() {
    return {
      loadExcel: false,
      loadPdf: false,
      loadPrint: false,
      checkAll: false,
      isIndeterminate: true,
      minChecked: 1,
      defaultCols: [],
      listDataTable: [],
      total: 0,
      listDataExport: [],
      page: PAGINATION_PARAM.page,
      size: PAGINATION_PARAM.size,
      filteredAllData: []
    }
  },
  computed: {
    /* Column select to Show/Hide*/
    renderTableColumns() {
      return this.columns && this.columns.length ? this.columns.filter((column) => column.show) : []
    },

    checkedTableColumns: {
      get() {
        return this.renderTableColumns.map(column => column.prop)
      },
      set(checked) {
        this.columns.forEach(column => {
          column.show = checked.includes(column.prop)
        })
      }
    },
    /* Column select to Show/Hide*/

    listLabel() {
      const listLabel = []
      this.renderTableColumns.forEach(column => {
        listLabel.push(column.label)
      })
      return listLabel
    },

    listProp() {
      const listProp = []
      this.renderTableColumns.forEach(column => {
        listProp.push(column.prop)
      })
      return listProp
    },

    syncData: {
      get() {
        return this.listDataTable
      },
      set(newData) {
        // this.$emit('update:listDataTable', newData)
        this.listDataTable = newData
      }
    },

    syncLoadData: {
      get() {
        return this.loading
      },
      set(val) {
        this.$emit('update:loading', val)
      }
    },

    syncTotal: {
      get() {
        return this.total
      },
      set(total) {
        // this.$emit('update:total', total)
        this.total = total
      }
    }
  },
  watch: {
    allData: {
      immediate: true,
      deep: true,
      handler: function(val, oldVal) {
        let allData
        if (this.filterEmptyRowData) {
          allData = val.filter(row => this.columns.map(column => row[column.prop]).every(data => data === 0 || !!data)) // null, undefined and '' will be treated as empty
        } else {
          allData = [...val]
        }
        this.filteredAllData = allData
        // move here

        let pageData
        if (this.isPagination) {
          this.page = 1
          pageData = this.filteredAllData.slice(0, this.size)
          this.syncTotal = this.filteredAllData.length
        } else {
          pageData = [...this.filteredAllData]
          this.syncTotal = this.filteredAllData.length
        }
        const sumRow = this.calculateSumRow()
        if (sumRow && this.filteredAllData.length) {
          pageData.push(sumRow)
        }
        this.syncData = pageData
      }
    },

  },
  mounted() {
    this.defaultCols = this.renderTableColumns.map(e => ({...e, isDefault: true}))
  },
  methods: {

    countStt(row, i) {
      if (row['sumTitleColumnSpan']) {
        return this.sumTitle
      } else {
        return ((this.page >= 1 ? this.page - 1 : 0) * this.size) + i + 1
      }
    },
    calSpan({row, column, rowIndex, columnIndex}) {
      if (row['sumTitleColumnSpan']) {
        console.log('total column index', row['sumTitleColumnIndex'], row['sumTitleColumnSpan'])
        if (columnIndex === row['sumTitleColumnIndex']) {
          return [1, row['sumTitleColumnSpan']]
        } else if (columnIndex < row['firstSumColumnIndex']) {
          return [0, 0]
        } else {
          return [1, 1]
        }
      } else {
        return [1, 1]
      }
    },
    calculateSumRow() {
      const sumColumnConfig = this.renderTableColumns.find(e => e['summarize'])
      let sumRow = this.filteredAllData.find(e => e['sumTitleColumnSpan'])
      if (sumColumnConfig) {
        const sumColumnIndex = this.renderTableColumns.indexOf(sumColumnConfig) + (this.showStt ? 1 : 0)
        if (!sumRow) {
          const newSumRow = this.renderTableColumns.reduce((obj, columnConfig) => {
            const propName = [columnConfig['prop']]
            if (columnConfig['summarize']) {
              const sumBigDeciaml = this.filteredAllData.reduce((sum, row) => {
                if (isNaN(parseFloat(row[propName]))) {
                  return sum
                }
                const val = new bigDecimal(row[propName])
                if (sum === null) {
                  sum = new bigDecimal(0)
                }
                return sum.add(val)
              }, null)
              if (sumBigDeciaml) {
                obj[propName] = sumBigDeciaml.getValue()
              }
            } else {
              obj[propName] = null
            }
            return obj
          }, {})
          newSumRow['firstSumColumnIndex'] = sumColumnIndex
          newSumRow['sumTitleColumnIndex'] = 0
          newSumRow['sumTitleColumnSpan'] = sumColumnIndex
          newSumRow[this.renderTableColumns[0]['prop']] = this.sumTitle
          sumRow = newSumRow
        }
      }
      return sumRow
    },
    /* Reset page to 0 when trigger onSearch()*/
    resetPage() {
      this.page = PAGINATION_PARAM.page
    },

    resetPageSize() {
      this.page = PAGINATION_PARAM.page
      this.size = PAGINATION_PARAM.size
    },
    pagination() {
      this.syncLoadData = true
      const pageIndex = this.page === 0 ? 0 : this.page - 1
      if (pageIndex > (this.syncTotal / this.size)) {
        return
      }
      const pageData = this.filteredAllData.slice(pageIndex * this.size, pageIndex * this.size + this.size)
      if (this.calculatedSumRow) {
        pageData.push(this.calculatedSumRow)
      }
      this.syncData = pageData
      this.syncTotal = this.filteredAllData.length
      setTimeout(() => {
        this.syncLoadData = false
      }, 200)
    },
    async exportExcel() {
      if (this.isExportTemplate === true) {
        this.checkIsValidApi(this.constantApiExcel, 'EXCEL [constant-api-excel]')
        this.loadExcel = true
        await apiFactory.download(this.constantApiExcel, this.fileName, this.constantApiExcel.params, this).then(() => {
          this.loadExcel = false
        }).catch(err => {
          this.loadExcel = false
          errAlert(this, err)
        })
      } else {
        this.loadExcel = true
        this.formatJson(this.listProp)
      }
    },

    formatJson(filterVal) {
      const param = deepClone(this.paramsFetch)
      const params = {
        ...param,
        page: 0,
        size: 5000
      }
      /* Xử lý nếu không truyền vào columns thì sẽ lấy columns tự defined. Không lấy cột STT và Thao tác*/
      let cols = this.renderTableColumns
      if (!cols.length) {
        cols = this.$refs.elTable.columns.filter(item => item.label !== 'STT' && item.label !== 'Thao tác')
        if (filterVal && !filterVal.length) {
          filterVal = cols.map(e => e.property)
        }
      }
      apiFactory.callAPI(this.apiFetch, {}, params).then(rs => {
        this.listDataExport = this.rsProp ? rs[this.rsProp].result : rs.result || []
        const execute = this.listDataExport.map((v, i) => filterVal.map((j, k) => {
          if (cols[k].formatter && typeof cols[k].formatter === 'function') {
            return cols[k].formatter(v, cols[k], v[j], i)
          } else {
            return v[j] ? v[j].toString() : ''
          }
        }))

        /* Xử lý Header excel nếu không truyền vào columns thì sẽ lấy header tự defined*/
        let labels = this.listLabel
        if (!labels.length) {
          labels = cols.map(e => e.label)
        }
        export_json_to_excel({
          header: labels,
          data: execute,
          // Lấy tên file là tên chức năng hiện tại
          filename: this.$route.meta.title,
          autoWidth: true,
          bookType: 'xlsx'
        })
        this.loadExcel = false
      }).catch(err => {
        this.loadExcel = false
        errAlert(this, err)
      })
    },

    async onGetDataPrint() {
      this.loadPrint = true
      this.checkIsValidApi(this.constantApiPdf, 'PDF [constant-api-pdf]')
      const {result} = await apiFactory.callAPI(this.constantApiPdf, {}, this.constantApiPdf.params)
      this.executeDataInList(result, this.renderTableColumns)
      this.loadPrint = false
      return result
    },

    executeDataInList(listData, listColumns) {
      listData.forEach((item, i) => {
        item.index = i + 1
        listColumns.forEach(col => {
          item[col['prop']] = checkDate(item[col['prop']]) ? formatFullDateTime_VN(item[col['prop']]) : replaceEscapeChar(item[col['prop']])
        })
      })
    },

    onExportPDF() {
      this.checkIsValidApi(this.constantApiPdf, 'PDF [constant-api-pdf]')
      this.loadPdf = true
      pdfMaker.vfs = pdfFonts.pdfMake.vfs
      if (this.arrsColIgnoreExport && this.arrsColIgnoreExport.length > 0) {
        this.listLabel = this.listLabel.filter(item => {
          return !this.arrsColIgnoreExport.includes(item)
        })
      }
      this.listProp = this.listProp.filter(item => {
        return item !== undefined
      })
      apiFactory.callAPI(this.constantApiPdf, {}, this.constantApiPdf.params).then(rs => {
        const lstDataFull = rs['result']
        lstDataFull.forEach((item, i) => {
          item.index = i + 1
        })
        const docDefinition = {
          footer(currentPage, pageCount) {
            return 'Trang ' + currentPage.toString() + '/' + pageCount
          },
          pageSize: this.pageSizePdf,
          pageOrientation: 'landscape',
          pageMargins: [40, 60, 40, 60],
          content: [
            {text: this.reportTitle, fontSize: 15, alignment: 'center', bold: true},
            {text: 'Từ ngày: ' + formatDate(this.dateFrom), alignment: 'center'},
            {text: 'Đến ngày: ' + formatDate(this.dateTo), alignment: 'center'},
            {
              width: 'auto',
              table: {
                headerRows: 1,
                widths: 'auto',
                body: []
              }
            }
          ]
        }
        const label = []
        this.listLabel.forEach(item => {
          label.push({text: item, alignment: 'center', bold: true, fillColor: '#dedede'})
        })
        docDefinition.content[3].table.body.push(label)

        let contentArrays
        for (let i = 0; i < lstDataFull.length; i++) {
          contentArrays = []
          this.listProp.forEach(item => {
            if (item.toLowerCase().includes('text')) {
              contentArrays.push({text: lstDataFull[i][item]})
            } else {
              contentArrays.push({text: checkDate(lstDataFull[i][item]) ? formatFullDateTime_VN(lstDataFull[i][item]) : lstDataFull[i][item]})
            }
          })
          docDefinition.content[3].table.body.push(contentArrays)
        }
        const fileName = this.fileName.substring(0, this.fileName.lastIndexOf('.'))
        pdfMaker.createPdf(docDefinition).download(`${fileName}_${moment(new Date()).format('dd_MM_yyyy')}.pdf`)
        this.loadPdf = false
      }).catch(err => {
        this.loadPdf = false
        errAlert(this, err)
      })
    },

    checkIsValidApi(constantApi, type) {
      if (!constantApi.url) {
        this.throwMsg('url ' + type)
      }
      if (!constantApi.method) {
        this.throwMsg('method ' + type)
      }
      if (!constantApi.params) {
        this.throwMsg('params ' + type)
      }
    },

    throwMsg(type) {
      // eslint-disable-next-line no-throw-literal
      throw new Error(`Chưa truyền ConstantAPI.${type}`)
    },

    handleCheckedAll(isAll) {
      if (isAll) {
        this.columns.map(column => {
          column.show = true
        })
      } else {
        this.columns.forEach(column => {
          column.show = this.defaultCols.filter(e => e.prop === column.prop).length > 0
        })
      }
      this.isIndeterminate = false
    },

    handleCheckedColumnsChange(value) {
      const checkedCount = value.length
      this.checkAll = checkedCount === this.columns.length
      this.isIndeterminate = checkedCount > 0 && checkedCount < this.columns.length
    },

    checkEscapeSequence(text, col) {
      const widthCol = document.getElementById('readMore')
      if (text.indexOf('\r\n') < 0 || text.indexOf('\n') < 0) {
        const firstRow = text.substring(0, text.indexOf('\n'))
        if (widthCol && firstRow.length > 25) {
          return firstRow.substring(0, 25).lastIndexOf(' ')
        }
        return text.indexOf('\n')
      }
      if (text.indexOf('\r\n') < text.indexOf('\n')) {
        return text.indexOf('\n')
      } else {
        return text.indexOf('\r')
      }
    },

    select(selection, row) {
      this.$emit('select', selection, row)
    },
    selectAll(val) {
      this.$emit('select-all', val)
    },
    selectionChange(val) {
      this.$emit('selection-change', val)
    },
    cellMouseEnter(row, column, cell, event) {
      this.$emit('cell-mouse-enter', row, column, cell, event)
    },
    cellMouseLeave(row, column, cell, event) {
      this.$emit('cell-mouse-leave', row, column, cell, event)
    },
    cellClick(row, column, cell, event) {
      this.$emit('cell-click', row, column, cell, event)
    },
    rowClick(row, column, event) {
      this.$emit('row-click', row, column, event)
    },
    rowContextmenu(row, column, event) {
      this.$emit('row-contextmenu', row, column, event)
    },
    rowDbclick(row, column, event) {
      this.$emit('row-dblclick', row, column, event)
    },
    headerClick(column, event) {
      this.$emit('header-click', column, event)
    },
    headerContextmenu(column, event) {
      this.$emit('header-contextmenu', column, event)
    },
    sortChange({column, prop, order}) {
      this.$emit('sort-change', {column, prop, order})
    },
    filterChange(filters) {
      this.$emit('filter-change', filters)
    },
    currentChange(currentRow, oldCurrentRow) {
      this.$emit('current-change', currentRow, oldCurrentRow)
    },
    headerDragend(newWidth, oldWidth, column, event) {
      this.$emit('header-dragend', newWidth, oldWidth, column, event)
    },
    expandChange(row, isExpanded) {
      this.$emit('expand-change', row, isExpanded)
    }
  }
}
moment.suppressDeprecationWarnings = true
</script>

<style lang="scss" scoped>
#readMore > > > span a {
  color: #1890ff;
}

#readMore {
  white-space: break-spaces;
}

#formUtil > > > .el-card__body {
  padding: 2px !important;
}

.utilStyle {
  margin-top: 5px
}

.checkedColumns {
  min-height: 250px;
  max-height: 300px;
  max-width: 500px;
  overflow: auto;
  margin: -8px;
  padding: 8px;
}

.client-table {
  ::v-deep thead .el-table__cell {
    font-size: 14px;
  }

  ::v-deep .el-table__empty-text {
    font-size: 14px;
  }
}
</style>
