<template>
  <el-dialog :visible="dialogVisible" :title="dialogTitle"
             @close="close" :close-on-click-modal="false"
             width="50%">
    <el-form>
      <el-row type="flex" justify="center">
        <el-col :span="!toImportProjectId ? 6 : 0">
          <div class="project-item">
            <div style="margin-bottom: 10px">
              {{ $t('project.select') }}
            </div>
            <el-select v-model="currentProjectId" filterable clearable>
              <el-option v-for="item in projectList" :key="item.id"
                         :label="item.name" :value="item.id">
              </el-option>
            </el-select>
          </div>
        </el-col>
        <el-col :span="16" style="text-align: center;">
          <el-upload
            class="api-upload" drag action="alert"
            :on-change="handleFileChange"
            :limit="1" :file-list="uploadFiles"
            :on-remove="handleRemove"
            :on-exceed="handleExceed"
            :auto-upload="false" accept=".json">
            <i class="el-icon-upload"></i>
            <div class="el-upload__text" v-html="$t('load_test.upload_tips')"></div>
            <div class="el-upload__tip" slot="tip">
              {{ $t('api_test.api_import.file_size_limit', {size: this.uploadSize}) }}
              {{ '，' + $t('api_test.api_import.ms_env_import_file_limit') }}
            </div>
          </el-upload>
        </el-col>
      </el-row>
    </el-form>
    <template v-slot:footer>
      <el-button type="primary" @click="save">
        {{ $t('commons.confirm') }}
      </el-button>
    </template>
  </el-dialog>
</template>

<script>
import {importEnvironment} from "../../../api/environment";
import {getCurrentProjectID} from "metersphere-frontend/src/utils/token";
import {getUploadSizeLimit} from "metersphere-frontend/src/utils/index";

export default {
  name: "EnvironmentImport",
  props: {
    projectList: {
      type: Array,
      default() {
        return [];
      }
    },
    toImportProjectId: {
      type: String,
      default() {
        return "";
      }
    }
  },
  data() {
    return {
      currentProjectId: '',   //所选中环境的id
      uploadFiles: [],
      dialogTitle: this.$t('api_test.environment.import'),
      dialogVisible: false
    }
  },
  watch: {
    //导入框激活时重置选中的项目和文件
    dialogVisible(val, oldVal) {
      if (oldVal === false) {
        this.currentProjectId = '';
        this.uploadFiles = [];
      }
    }
  },
  computed: {
    uploadSize() {
      return getUploadSizeLimit();
    }
  },

  methods: {
    handleFileChange(file, uploadFiles) {
      this.uploadFiles = uploadFiles;
    },
    save() {
      if (this.uploadFiles.length > 0) {
        for (let i = 0; i < this.uploadFiles.length; i++) {
          if (this.uploadValidate(this.uploadFiles[i])) {
            return;
          }
          let file = this.uploadFiles[i];
          if (!file) {
            continue;
          }
          let reader = new FileReader();
          reader.readAsText(file.raw)
          reader.onload = (e) => {
            try {
              let arr = [];
              JSON.parse(e.target.result).map(env => {
                env.projectId = getCurrentProjectID();
                arr.push(env);
              })
              importEnvironment(arr).then(res => {
                if (res.data === 'OK') {
                  this.$success(this.$t('commons.save_success'));
                } else {
                  this.$success(this.$t('commons.save_success') + this.$t('pj.environment_import_repeat_tip', [res.data]));
                }
                this.dialogVisible = false;
                this.$emit('refresh');
              });
            } catch (exception) {
              this.$warning(this.$t('api_test.api_import.ms_env_import_file_limit'));
            }
          }
        }
      }
    },
    handleExceed() {
      this.$warning(this.$t('api_test.api_import.file_exceed_limit'));
    },
    handleRemove() {

    },
    uploadValidate(file) {    //判断文件扩展名是不是.json，以及文件大小是否超过系统限制
      const extension = file.name.substring(file.name.lastIndexOf('.') + 1);
      if (!(extension === 'json')) {
        this.$warning(this.$t('api_test.api_import.ms_env_import_file_limit'));
        return true;
      }
      if (file.size / 1024 / 1024 > this.uploadSize) {
        this.$warning(this.$t('api_test.api_import.file_size_limit', {size: this.uploadSize}));
        return true
      }
      return false;
    },
    open() {
      this.dialogVisible = true;
    },
    close() {
      this.dialogVisible = false;
    }
  },

}
</script>

<style scoped>
.project-item {
  padding-left: 20px;
  padding-right: 20px;
}
</style>
