<template>
  <div v-loading="result.loading">
    <el-card class="table-card">
      <template v-slot:header>
        <div>
          <el-row class="table-title" type="flex" justify="space-between" align="middle">
            <span class="title">API Keys</span>
          </el-row>
          <el-row type="flex" justify="space-between" align="middle">
            <el-button @click="createApiKey()" plain type="el-icon-question" icon="el-icon-circle-plus-outline"
                       size="mini">
              {{ $t('commons.create') }}
            </el-button>
          </el-row>
        </div>
      </template>

      <el-table border class="adjust-table" :data="tableData" style="width: 100%">
        <el-table-column prop="accessKey" label="Access Key">
          <template v-slot:default="scope">
            <div class="variable-combine">
              <div class="variable">{{ scope.row.accessKey }}</div>
              <div>
                <el-tooltip :content="$t('api_test.copied')" manual v-model="scope.row.visible" placement="top"
                            :visible-arrow="false">
                  <i class="el-icon-copy-document copy" @click="copy(scope.row, 'accessKey', 'visible')"/>
                </el-tooltip>
              </div>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="secretKey" label="Secret Key">
          <template v-slot:default="scope">
            <el-link type="primary" @click="showSecretKey(scope)" v-if="!scope.row.apiKeysVisible">{{ $t('commons.show') }}</el-link>
            <div v-if="scope.row.apiKeysVisible" class="variable-combine">
              <div class="variable">{{scope.row.secretKey}}</div>
                <el-tooltip :content="$t('api_test.copied')" manual v-model="scope.row.visible2" placement="top"
                            :visible-arrow="false">
                  <i class="el-icon-copy-document copy" @click="copy(scope.row, 'secretKey', 'visible2')"/>
                </el-tooltip>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="status" :label="$t('commons.status')">
          <template v-slot:default="scope">
            <el-switch v-model="scope.row.status"
                       inactive-color="#DCDFE6"
                       active-value="ACTIVE"
                       inactive-value="DISABLED"
                       @change="changeSwitch(scope.row)"
            />
          </template>
        </el-table-column>
        <el-table-column prop="createTime" :label="$t('commons.create_time')">
          <template v-slot:default="scope">
            <span>{{ scope.row.createTime | timestampFormatDate }}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('commons.operating')">
          <template v-slot:default="scope">
            <div>
              <ms-table-operator-button :tip="$t('commons.delete')" icon="el-icon-delete"
                                        type="danger" @exec="deleteApiKey(scope.row)"/>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
import MsDialogFooter from "../../common/components/MsDialogFooter";
import {getCurrentUser, operationConfirm} from "@/common/js/utils";
import MsTableOperatorButton from "../../common/components/MsTableOperatorButton";
import MsTableHeader from "../../common/components/MsTableHeader";

export default {
  name: "MsApiKeys",
  components: {MsDialogFooter, MsTableOperatorButton, MsTableHeader},
  data() {
    return {
      result: {},
      updateVisible: false,
      editPasswordVisible: false,
      apiKeysVisible: false,
      condition: {},
      tableData: [],
      currentRow: {},
    }
  },

  created() {
    this.search();
  },

  methods: {
    currentUser: () => {
      return getCurrentUser();
    },
    search() {
      this.result = this.$get("/user/key/info", response => {
          response.data.forEach((d) => {
            d.show = false;
            d.apiKeysVisible = false;
          })
          this.tableData = response.data;
        }
      )
    },
    deleteApiKey(row) {
      operationConfirm(this, this.$t('user.apikey_delete_confirm'), () => {
        this.result = this.$get("/user/key/delete/" + row.id, response => {
          this.$success(this.$t('commons.delete_success'));
          this.search();
        })
      });
    },

    createApiKey() {
      this.result = this.$get("/user/key/generate", response => {
        this.$success(this.$t('commons.save_success'));
        this.search();
      })
    },

    changeSwitch(row) {
      if (row.status === 'ACTIVE') {
        this.result = this.$get("/user/key/active/" + row.id, response => {
          this.$success(this.$t('commons.save_success'));
        });
      }
      if (row.status === 'DISABLED') {
        this.result = this.$get("/user/key/disable/" + row.id, response => {
          this.$success(this.$t('commons.save_success'));
        });
      }
    },
    showSecretKey(scope) {
      scope.row.apiKeysVisible = true
      setTimeout(() => {
        //this.$set(this.tableData[row.$index], "apiKeysVisible", false);
        scope.row.apiKeysVisible = false
      }, 5000);
    },
    copy(row, key, visible) {
      let input = document.createElement("input");
      document.body.appendChild(input);
      input.value = row[key];
      input.select();
      if (input.setSelectionRange) {
        input.setSelectionRange(0, input.value.length);
      }
      document.execCommand("copy");
      document.body.removeChild(input);
      row[visible] = true;
      setTimeout(() => {
        row[visible] = false;
      }, 1000);
    },
  }
}
</script>

<style scoped>
.variable-combine {
  color: #7F7F7F;
  line-height: 32px;
  position: absolute;
  top: 10px;
  margin-right: -20px;
  display: flex;
  align-items: center;
}

.variable {
  display: inline-block;
  margin-right: 10px;
  overflow-x: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.copy {
  font-size: 14px;
  cursor: pointer;
  color: #1E90FF;
}
</style>
