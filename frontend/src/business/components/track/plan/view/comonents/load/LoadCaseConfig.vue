<template>
  <el-dialog :close-on-click-modal="false" :visible.sync="visible" :title="$t('load_test.pressure_config')"
             width="85%" top="8vh" @close="close" v-loading="result.loading" :destroy-on-close="true">
    <performance-load-config :test-id="loadTestId" :load-case-id="loadCaseId" :resource-pool="poolId"
                             @fileChange="fileChange" ref="pressureConfig" style="height: 50vh; overflow-y: auto;"/>
    <template v-slot:footer>
      <el-button @click="close" size="medium">{{ $t('commons.cancel') }}</el-button>
      <el-button type="primary" @click="onSubmit" size="medium" style="margin-left: 10px;">
        {{ $t('commons.confirm') }}
      </el-button>
    </template>
  </el-dialog>
</template>

<script>

import PerformanceLoadConfig from "@/business/components/track/plan/view/comonents/load/PerformanceLoadConfig";

export default {
  name: "LoadCaseConfig",
  components: {
    PerformanceLoadConfig
  },
  data() {
    return {
      visible: false,
      loadTestId: "",
      result: {},
      loadCaseId: "",
      poolId: ""
    }
  },
  methods: {
    open(loadTestId, loadCaseId) {
      this.visible = true;
      this.loadTestId = loadTestId;
      this.loadCaseId = loadCaseId;
      this.result = this.$get("/test/plan/load/case/get/" + loadCaseId, res => {
        this.poolId = res.data ? res.data.testResourcePoolId : "";
      })
    },
    close() {
      this.visible = false;
    },
    fileChange(threadGroups) {
      let handler = this.$refs.pressureConfig;

      let csvSet = new Set;
      threadGroups.forEach(tg => {
        tg.threadNumber = tg.threadNumber || 10;
        tg.duration = tg.duration || 10;
        tg.rampUpTime = tg.rampUpTime || 5;
        tg.step = tg.step || 5;
        tg.rpsLimit = tg.rpsLimit || 10;
        tg.threadType = tg.threadType || 'DURATION';
        tg.iterateNum = tg.iterateNum || 1;
        tg.iterateRampUp = tg.iterateRampUp || 10;

        if (tg.csvFiles) {
          tg.csvFiles.map(item => csvSet.add(item));
        }
      });

      this.$set(handler, "threadGroups", threadGroups);
      this.$refs.pressureConfig.threadGroups = threadGroups;

      handler.calculateTotalChart();
    },
    onSubmit() {
      if (!this.$refs.pressureConfig.validConfig()) {
        return false;
      }
      // 压力配置
      let loadConfiguration = JSON.stringify(this.$refs.pressureConfig.convertProperty());
      let testResourcePoolId = this.$refs.pressureConfig.resourcePool;
      let params = {
        id: this.loadCaseId,
        loadConfiguration,
        testResourcePoolId
      }
      this.result = this.$post("/test/plan/load/case/update", params, () => {
        this.visible = false;
        this.$success(this.$t("commons.modify_success"));
      })
    },
  }
}
</script>

<style scoped>

</style>
