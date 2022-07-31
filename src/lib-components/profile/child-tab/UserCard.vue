<template>
  <el-card style="margin-bottom:20px;">
    <div slot="header" class="clearfix">
      <span>THÔNG TIN</span>
    </div>

    <div class="userInfo-profile">
      <div class="box-center">
        <pan-thumb
          :height="'150px'"
          :hoverable="false"
          :width="'150px'"
        >
          <div>Xin chào</div>
          {{ userInfo.ufn }}
        </pan-thumb>
      </div>
      <div class="box-center">
        <div class="userInfo-name text-center">{{ userInfo.ufn }}</div>
      </div>
    </div>

    <div class="userInfo-bio">
      <div class="userInfo-skills userInfo-bio-section">
        <div class="userInfo-bio-section-header">
          <svg-icon icon-class="skill"/>
          <span>{{ userInfo.orgName }}</span>
        </div>

        <div class="userInfo-bio-section-body">
          <el-form
            ref="form"
            label-width="90px"
          >
            <el-form-item label="Phòng ban:">
              <span>{{ userInfo.depName }}</span>
            </el-form-item>
            <el-form-item label="Chức vụ:">
              <span>{{ userInfo.posName }}</span>
            </el-form-item>
          </el-form>
        </div>
      </div>
    </div>
  </el-card>
</template>

<script>
import PanThumb from '../../../components/PanThumb/index.vue'
import {mapGetters} from 'vuex'

export default {
  name: 'UserCard',
  components: { PanThumb },
  computed: {
    ...mapGetters(['userInfo'])
  }
}
</script>

<style lang="scss" scoped>
.box-center {
  margin: 0 auto;
  display: table;
}

.text-muted {
  color: #777;
}

.userInfo-profile {
  .userInfo-name {
    font-weight: bold;
  }

  .box-center {
    padding-top: 10px;
  }

  .userInfo-role {
    padding-top: 10px;
    font-weight: 400;
    font-size: 14px;
  }

  .box-social {
    padding-top: 30px;

    .el-table {
      border-top: 1px solid #dfe6ec;
    }
  }

  .userInfo-follow {
    padding-top: 20px;
  }
}

.userInfo-bio {
  margin-top: 20px;
  color: #606266;

  span {
    padding-left: 4px;
  }

  .userInfo-bio-section {
    font-size: 14px;
    padding: 15px 0;

    .userInfo-bio-section-header {
      border-bottom: 1px solid #dfe6ec;
      padding-bottom: 10px;
      margin-bottom: 10px;
      font-weight: bold;
      text-align: center;
    }
  }
}
</style>
