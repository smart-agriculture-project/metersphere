<template>
  <el-dialog :visible="visible" :title="$t('test_track.issue.sync_bugs')" @close="cancel">
    <el-form ref="form" :model="form" :rules="rules" label-width="300px" :inline="true" size="small">
      <el-form-item prop="typeValue">
        <el-select v-model="form.typeValue">
          <el-option
            v-for="item in form.timeTypeOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item prop="preValue">
        <el-select v-model="form.preValue">
          <el-option
            v-for="item in form.preOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item prop="createTime">
        <el-date-picker
          v-model="form.createTime"
          type="datetime"
          @change="changeSyncTime"
          placeholder="选择日期时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item class="tips-item">
        <span class="tips">{{ $t('custom_field.sync_issue_tips') }}</span>
      </el-form-item>
    </el-form>

    <template v-slot:footer>
      <div class="dialog-footer">
        <el-button size="small" @click="cancel">{{ $t('commons.cancel') }}</el-button>
        <el-button type="primary" size="small" @click="save">{{ $t('commons.confirm') }}</el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script>
export default {
  name: "IssueSyncSelect",
  data() {
    return {
      visible: false,
      form: {
        timeTypeOptions: [{"value": "createTime", "label": "创建时间"}],
        typeValue: "createTime",
        preOptions: [{"value": false, "label": "大于等于"}, {"value": true, "label": "小于等于"}],
        preValue: false,
        createTime: ""
      },
      rules: {
        createTime: [
          { type: 'date', required: true, message: "请选择日期时间", trigger: 'change'}
        ]
      }
    }
  },
  methods: {
    open() {
      this.visible = true;
      let syncTime = localStorage.getItem("ISSUE_SYNC_TIME");
      if (syncTime) {
        this.form.createTime = new Date(syncTime);
      }
    },
    save() {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          this.visible = false;
          this.$emit('syncConfirm', this.form);
        } else {
          return false;
        }
      });
    },
    cancel() {
      this.visible = false;
    },
    changeSyncTime() {
      if (this.form.createTime) {
        localStorage.setItem("ISSUE_SYNC_TIME", this.form.createTime);
      }
    }
  }
};
</script>

<style scoped>
.tips{
  font-size: 13px;
}

.el-form-item.tips-item.el-form-item--small {
  top: -20px;
  left: 10px;
}

.el-form-item.btn-group.el-form-item--small {
  float: right;
  position: relative;
  top: 25px;
}
</style>
