<template>
  <div class="app-container">
    <div>
      <el-row :gutter="20">
        <el-col :span="6" :xs="24">
          <user-card />
        </el-col>

        <el-col :span="18" :xs="24">
          <el-card>
            <el-tabs v-model="activeTab" @tab-click="onTabClick">
              <el-tab-pane :name="defaultTab" label="Thông tin cá nhân">
                <user-information />
              </el-tab-pane>

              <!-- <el-tab-pane v-loading="loading" label="Lịch sử điều chuyển công tác" name="transfer">
                <transfers :data="listTransfer" style="margin-left: -3% !important;"/>
              </el-tab-pane> -->

              <el-tab-pane label="Đổi mật khẩu" name="account">
                <account :active-tab.sync="activeTab" />
              </el-tab-pane>

              <!-- <el-tab-pane label="Kích hoạt xác thực hai yếu tố OTP" name="twoFactor">
                <two-factor/>
              </el-tab-pane> -->
            </el-tabs>
          </el-card>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
import UserCard from "./child-tab/UserCard.vue";
import UserInformation from "./child-tab/UserInformation.vue";
import Account from "./child-tab/Account.vue";
import TwoFactor from "./child-tab/TwoFactor.vue";
import Transfers from "../../components/TransferOrg/index.vue";
import apiFactory from "ff24-js/src/api/apiFactory";
import ConstantAPI from "ff24-js/src/utils/ConstantAPI";
import { errAlert } from "ff24-js/src/utils/AlertMessage";

const DEFAULT_TAB = "userInformation";

export default {
  name: "Profile",
  components: { UserInformation, UserCard, Account, TwoFactor, Transfers },
  data() {
    return {
      defaultTab: DEFAULT_TAB,
      activeTab: DEFAULT_TAB,
      loading: false,
      listTransfer: [],
    };
  },
  methods: {
    onTabClick({ name }) {
      if (name === "transfer") {
        this.loading = true;
        apiFactory
          .callAPI(ConstantAPI["ql-nguoi-dung"].TRANSFER_ORG)
          .then((rs) => {
            this.listTransfer = rs;
          })
          .catch((err) => {
            console.log("Lỗi: onTabClick: Profile.vue");
            errAlert(this, err);
          })
          .finally(() => {
            this.loading = false;
          });
      }
    },
  },
};
</script>
