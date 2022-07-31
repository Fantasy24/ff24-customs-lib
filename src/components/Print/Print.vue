<template>
  <span>
    <el-button class="printer" circle size="medium" title="In" type="text" @click="onPrint">
      <em class="el-icon-printer" />
    </el-button>
    <span style="display: none">
      <div :id="id" class="print">
        <table border="0" cellpadding="0" cellspacing="0" width="100%">
          <tr>
            <td>TỔNG CỤC HẢI QUAN</td>
            <td />
            <td />
          </tr>
          <tr>
            <td align="center" colspan="3">
              <strong>{{ exportTitle }}</strong>
            </td>
          </tr>
          <tr>
            <td align="center" colspan="3">
              <em>
                Từ ngày: {{ dateFrom | formatDate }} đến ngày:
                {{ dateTo | formatDate }}
              </em>
            </td>
          </tr>
        </table>
        <table
          id="data"
          border="0"
          cellpadding="0"
          cellspacing="0"
          style="
            -webkit-print-color-adjust: exact;
            border-left: 1px solid black;
            border-top: 1px solid black;
          "
          width="100%"
        >
          <thead>
            <tr style="background-color: #dedede">
              <td
                v-for="col in cols"
                :key="col.prop"
                :width="col.width || 150"
                align="center"
              >
                <strong>{{ col.text }}</strong>
              </td>
            </tr>
          </thead>
          <tbody>
            <tr v-for="row in dataSource" :key="row.STT">
              <td v-for="col in cols" :key="col.prop" :align="col.alignment">
                {{ row[col.prop] }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </span>
  </span>
</template>

<script>
import Printd from 'printd';
import _ from 'lodash';
import {date} from 'jszip/lib/defaults';

export default {
  name: 'Print',
  props: {
    dateFrom: {
      type: date,
      default: null
    },
    exportTitle: {
      type: String,
      default: ''
    },
    dateTo: {
      type: date,
      default: null
    },
    columns: {
      type: Array,
      default: () => {
      }
    },
    remoteMethod: {
      type: Function,
      default: null
    }
  },
  data() {
    return {
      dataSource: [],
      // eslint-disable-next-line vue/no-reserved-keys
      _cols: [],
      id: null
    }
  },
  computed: {
    cols: {
      get() {
        return _.map(this.columns, (item) => {
          return {
            prop: item.prop,
            text: item.label,
            alignment: item.align ? item.align.replace('is-', '') : 'left',
            width: item.width || 200,
            bold: true,
            fillColor: '#dedede'
          }
        }).filter((x) => {
          return x.prop
        })
      }
    }
  },
  mounted() {
    this.id = `print_${this._uid}`
  },
  methods: {
    async onPrint() {
      if (this.remoteMethod) this.dataSource = await this.remoteMethod()
      this.$nextTick(function() {
        const d = new Printd()

        d.print(document.getElementById(this.id), [
          `@page {size: A4 landscape;}
          #data td { border-bottom: 1px solid black; border-right: 1px solid black; }`
        ])
      })
    }
  }
}
</script>
