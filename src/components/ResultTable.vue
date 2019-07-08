<template>
  <el-table border height="250" :data="items" class="input-table">
    <el-table-column
      :key="`col-${index}`"
      v-for="(column, index) in columnsReward"
      :label="column.name"
    >
      <template slot-scope="scope">{{ scope.row[column.index] }}</template>
    </el-table-column>
  </el-table>
</template>

<script>
export default {
  name: "result-table",
  props: {
    items: {
      type: Array,
      default: () => []
    },
    columns: {
      type: Array,
      default: () => []
    },
    rawColumns: {
      type: Array,
      default: () => []
    }
  },
  computed: {
    columnsReward() {
      if (this.columns.length > 0) {
        return [
          ...this.columns,
          {
            index: this.rawColumns[this.rawColumns.length - 1].index + 1,
            name: "รางวัล"
          },
          {
            index: this.rawColumns[this.rawColumns.length - 1].index + 2,
            name: "รางวัลที่"
          }
        ];
      } else {
        return [];
      }
    }
  }
};
</script>

