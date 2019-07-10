<template>
  <el-table border height="250" :data="items" class="input-table">
    <el-table-column
      :key="`col-${index}`"
      v-for="(column, index) in columnsReward"
      :label="column.name"
    >
      <template slot-scope="scope">{{ scope.row[index] }}</template>
      <!-- <template slot-scope="scope">{{items }}</template> -->
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
        let newIndexColumn = JSON.parse(JSON.stringify(this.columns));
        newIndexColumn.forEach(column => {
          column.index = column.index + 2;
        });
        return [
          {
            index: 0,
            name: "รางวัลที่"
          },
          {
            index: 1,
            name: "รางวัล"
          },
          ...newIndexColumn
        ];
      } else {
        return [];
      }
    }
  }
};
</script>

