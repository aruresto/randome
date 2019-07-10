<template>
  <el-table border height="250" :data="inputs" class="input-table">
    <el-table-column :key="`col-${index}`" v-for="(column, index) in columns" :label="column.name">
      <template slot-scope="scope">{{ scope.row[index] }}</template>
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
    },
    columns: {
      type: Array,
      default: () => []
    }
  },
  computed: {
    noInput() {
      return this.inputs.length === 0;
    }
  }
};
</script>

<style lang="scss" scoped>
.input-table {
  width: 100%;
}
</style>

