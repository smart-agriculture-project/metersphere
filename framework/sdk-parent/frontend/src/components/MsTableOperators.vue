<template>
  <div class="operator-btn-group">
    <ms-table-operator-button
      v-for="(btn, index) in buttons"
      :key="index"
      v-permission="btn.permissions"
      :disabled="isDisable(btn)"
      :class="btn.class"
      :row-data="row"
      :tip="tip(btn)"
      :icon="btn.icon"
      :type="btn.type"
      :isDivButton="btn.isDivButton"
      :is-text-button="btn.isTextButton"
      :is-more-operate="btn.isMoreOperate"
      :child-operate="btn.childOperate"
      @exec="click(btn)"
      @click.stop="clickStop(btn)"
    />
  </div>
</template>

<script>
import MsTableOperatorButton from "./MsTableOperatorButton";

export default {
  name: "MsTableOperators",
  components: { MsTableOperatorButton },
  props: {
    row: Object,
    buttons: Array,
    index: Number,
  },
  methods: {
    click(btn) {
      if (btn.exec instanceof Function) {
        btn.exec(this.row, this.index);
      }
    },
    clickStop(btn) {
      if (btn.stop instanceof Function) {
        btn.stop(this.row, this.index);
      }
    },
    isDisable(btn) {
      if (btn.isDisable) {
        if (btn.isDisable instanceof Function) {
          return btn.isDisable(this.row);
        } else {
          return btn.isDisable;
        }
      }
      return false;
    },
    tip(btn) {
      if (btn.tip instanceof Function) {
        return btn.tip(this.row);
      } else {
        return btn.tip;
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.operator-btn-group {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  margin-right: 8px;
  .el-button {
    margin-right: 0;
  }
}
</style>
