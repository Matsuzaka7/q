<template>
  <div>
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="50%"
      :close-on-click-modal="false"
      append-to-body
      :modal-append-to-body="false"
    >
      <el-table :data="tableData" style="width: 100%">
        <el-table-column prop="name" label="名称"> </el-table-column>
        <el-table-column label="操作" width="200">
          <template slot-scope="{ row }">
            <el-button
              size="mini"
              @click="handleTop(row)"
              type="primary"
              plain
              icon="el-icon-caret-top"
            ></el-button>
            <el-button
              size="mini"
              type="primary"
              @click="handleBottom(row)"
              plain
              icon="el-icon-caret-bottom"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleClick">保 存</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "SortDialog",
  props: ["sortData"],
  data() {
    return {
      dialogVisible: false,
      tableData: [],
    };
  },
  watch:{
    sortData(){
      this.dataTrans(this.sortData);
    }
  },
  methods: {
    openDialog() {
      this.dialogVisible = true;
    },
    handleTop(row) {
      const arr = this.sortData;
      //上移
      const index = arr.indexOf(row.name);
      if (index != -1 && index != 0) {
        let mid = arr[index];
        arr[index] = arr[index - 1];
        arr[index - 1] = mid;
        this.dataTrans(arr);
      }
    },
    handleBottom(row) {
      const arr = this.sortData;
      //下移
      const index = arr.indexOf(row.name);
      if (index != -1 && index + 1 != arr.length) {
        let mid = arr[index];
        arr[index] = arr[index + 1];
        arr[index + 1] = mid;
        this.dataTrans(arr);
      }
    },
    dataTrans(sortData) {
      const arr = [];
      sortData.forEach((item) => {
        arr.push({
          name: item,
        });
      });
      this.tableData = arr;
    },
    handleClick(){
      this.$emit('updateData',this.tableData.map(x=>x.name))
      this.dialogVisible = false
    },
  },
};
</script>

<style>
</style>