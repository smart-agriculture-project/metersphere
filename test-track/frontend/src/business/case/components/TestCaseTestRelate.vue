<template>
  <div>

    <el-dropdown  @command="handleCommand" :disabled="readOnly">
      <el-button type="primary" size="mini" :disabled="readOnly">
        {{$t('test_track.case.relate_test')}}<i class="el-icon-arrow-down el-icon--right"></i>
      </el-button>
      <el-dropdown-menu slot="dropdown">
        <el-dropdown-item command="api">{{ $t('api_test.home_page.failed_case_list.table_value.case_type.api') }}</el-dropdown-item>
        <el-dropdown-item command="scenario">{{ $t('api_test.home_page.failed_case_list.table_value.case_type.scene') }}</el-dropdown-item>
        <el-dropdown-item command="ui" v-xpack>{{ $t('api_test.home_page.failed_case_list.table_value.case_type.ui') }}</el-dropdown-item>
        <el-dropdown-item command="performance">{{$t('api_test.home_page.failed_case_list.table_value.case_type.load')}}</el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>

    <ms-table
      v-loading="result.loading"
      :show-select-all="false"
      :data="data"
      :enable-selection="false"
      :operators="operators"
      @refresh="initTable">

      <ms-table-column
        prop="num"
        label="ID">
      </ms-table-column>

      <ms-table-column
        prop="name"
        :label="$t('test_track.case.name')"/>

      <ms-table-column
        prop="projectName"
        :label="$t('commons.project')"/>

      <ms-table-column
        v-if="versionEnable"
        prop="versionName"
        :label="$t('commons.version')"/>

      <ms-table-column
        :label="$t('test_resource_pool.type')"
        prop="type">
        <template v-slot:default="{row}">
          {{typeMap[row.testType]}}
        </template>
      </ms-table-column>
    </ms-table>

    <test-case-api-relate
      :case-id="caseId"
      :versionEnable="versionEnable"
      :not-in-ids="notInIds"
      @refresh="initTable"
      ref="apiCaseRelevance"/>

    <test-case-scenario-relate
      :case-id="caseId"
      :versionEnable="versionEnable"
      :not-in-ids="notInIds"
      @refresh="initTable"
      ref="apiScenarioRelevance"/>

    <test-case-ui-scenario-relate
      :case-id="caseId"
      :versionEnable="versionEnable"
      :not-in-ids="notInIds"
      @refresh="initTable"
      ref="uiScenarioRelevance"/>

    <test-case-load-relate
      :case-id="caseId"
      :not-in-ids="notInIds"
      :versionEnable="versionEnable"
      @refresh="initTable"
      ref="loadRelevance"/>

  </div>
</template>

<script>
import MsTable from "metersphere-frontend/src/components/table/MsTable";
import MsTableColumn from "metersphere-frontend/src/components/table/MsTableColumn";
import TestCaseApiRelate from "@/business/case/components/TestCaseApiRelate";
import {deleteRelateTest, getRelateTest} from "@/api/testCase";
import TestCaseScenarioRelate from "@/business/case/components/TestCaseScenarioRelate";
import TestCaseUiScenarioRelate from "@/business/case/components/TestCaseUiScenarioRelate";
import TestCaseLoadRelate from "@/business/case/components/TestCaseLoadRelate";
import TestCaseUiScenarioRelevance from "@/business/plan/view/comonents/ui/TestCaseUiScenarioRelevance";

export default {
  name: "TestCaseTestRelate",
  components: {
    TestCaseUiScenarioRelevance,
    TestCaseLoadRelate, TestCaseScenarioRelate, TestCaseApiRelate, MsTableColumn, MsTable, TestCaseUiScenarioRelate},
  data() {
    return {
      data: [],
      result: {},
      typeMap: {
        testcase: this.$t('api_test.home_page.failed_case_list.table_value.case_type.api'),
        automation: this.$t('api_test.home_page.failed_case_list.table_value.case_type.scene'),
        performance: this.$t('api_test.home_page.failed_case_list.table_value.case_type.load'),
        uiAutomation: this.$t('api_test.home_page.failed_case_list.table_value.case_type.ui')
      },
      operators: [
        {
          tip: this.$t('test_track.plan_view.cancel_relevance'), icon: "el-icon-unlock",
          exec: this.remove,
          type: 'danger',
          isDisable: () => {
            return this.readOnly;
          }
        }
      ],
      notInIds: null
    }
  },
  props: ['caseId', 'readOnly', 'versionEnable'],
  methods: {
    handleCommand(key) {
      if (!this.caseId) {
        this.$warning(this.$t('api_test.automation.save_case_info'));
        return;
      }
      if (key === 'api') {
        this.$refs.apiCaseRelevance.open();
      } else if (key === 'scenario') {
        this.$refs.apiScenarioRelevance.open();
      } else if (key === 'performance') {
        this.$refs.loadRelevance.open();
      } else if (key === 'ui') {
        this.$refs.uiScenarioRelevance.open();
      }
    },
    remove(row) {
      deleteRelateTest(row.testCaseId, row.testId)
        .then(() => {
          this.$success(this.$t('commons.save_success'));
          this.initTable();
        });
    },
    initTable() {
      if (this.caseId) {
        getRelateTest(this.caseId)
          .then((response) => {
            this.data = response.data;
            this.notInIds = this.data.map(i => i.testId);
          });
      }
    },
  }
}
</script>

<style scoped>
</style>
