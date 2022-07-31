<template>
  <div class="block">
    <div v-if="data && data.length">
      <el-timeline>
        <el-timeline-item
            v-for="transfer in data"
            :key="transfer.username"
            :timestamp="`Ngày điều chuyển: ${formatFullDateTime_VN(transfer.createdDate)}`"
            placement="top"
            type="success"
        >
          <el-card>
            <el-form label-width="140px">
              <el-row :gutter="10">
                <el-col :lg="12" :md="24" :sm="24" :xl="12" :xs="24">
                  <el-form-item label="Được chuyển từ">
                    <el-tag effect="plain" size="default" type="info" v-text="transfer.orgCodeOldText"/>
                  </el-form-item>
                </el-col>
                <el-col :lg="12" :md="24" :sm="24" :xl="12" :xs="24">
                  <el-form-item label="Đến" label-width="auto">
                    <el-tag effect="plain" size="default" type="success" v-text="transfer.orgCodeNewText"/>
                  </el-form-item>
                </el-col>
              </el-row>
              <el-form-item label="Ghi chú">
                <span v-html="transfer.description.replaceAll('\n','<br/>')"/>
              </el-form-item>
            </el-form>
            <div class="bottom clearfix">
            <span style="font-style: italic;float: right">
              Người điều chuyển: {{ transfer.createdUser }}
            </span>
            </div>
          </el-card>
        </el-timeline-item>
      </el-timeline>
    </div>
    <div v-else style="text-align: center">
      <h3>Người dùng chưa có thông tin điều chuyển công tác</h3>
    </div>
  </div>
</template>

<script>
import {formatFullDateTime_VN} from 'ff24-js/src/filters'

export default {
  props: {
    data: Array,
  },
  methods: {
    formatFullDateTime_VN
  }
}
</script>

<style scoped>

</style>
