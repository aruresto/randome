<template>
  <el-table border height="250" :data="inputs" class="input-table">
    <el-table-column :label="`รางวัลที่`">
      <template slot-scope="scope">{{ scope.$index + 1 }}</template>
    </el-table-column>
    <el-table-column :label="`รางวัล`">
      <template slot-scope="scope">{{ scope.row }}</template>
    </el-table-column>
  </el-table>
</template>

<script>
export default {
  name: "input-table",
  props: {
    inputs: {
      type: Array,
      default: () => []
    }
  }
};
</script>

<style lang="scss" scoped>
.input-table {
  width: 100%;
}
</style>

