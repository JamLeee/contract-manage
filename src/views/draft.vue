<template>
  <div class="app-container">
    <d-search @search="onsearch"></d-search>
    <el-table
      v-loading="listLoading"
      ref="filterTable"
      :data="tableData"
      border
      style="width: 100%"
    >
      <el-table-column align="center" prop="id" label="排序" width="50"></el-table-column>
      <el-table-column
        align="center"
        prop="contractType"
        label="合同类型"
        width="100"
        :formatter="formatter"
      ></el-table-column>
      <el-table-column align="center" prop="contractNumber" label="合同编号" width="100"></el-table-column>
      <el-table-column align="center" prop="partyA" label="甲方" width="100"></el-table-column>
      <el-table-column align="center" prop="createTime" label="合同创建日期" width="180"></el-table-column>
      <el-table-column align="center" prop="operator" label="合同操作者" width="100"></el-table-column>
      <el-table-column align="center" prop="lastModifyTime" label="最后操作时间" width="200"></el-table-column>
      <el-table-column align="center" label="操作" width="220" v-if="hasPermisson">
        <template slot-scope="props">
          <el-button size="mini" type="danger" @click="edit(props.row.id)">编辑</el-button>
          <el-button size="mini" type="success" @click="submit(props.row.id)">提交</el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />
  </div>
</template>

<script>
import Pagination from "@/components/Pagination"; // Secondary package based on el-pagination
import dSearch from '@/components/d-search.vue'
import contractMapObj from '@/utils/data/contractMapObj.json' // 映射的合同

export default {
  data() {
    return {
      tableData: [],
      total: 0,
      listQuery: {
        page: 1,
        limit: 20
      },
      hasPermisson: true, // 是否有权限显示按钮
      listLoading: true, // 表格loading
      pickerOptions: {
        shortcuts: [
          {
            text: "最近一周",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", [start, end]);
            }
          },
          {
            text: "最近一个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit("pick", [start, end]);
            }
          },
          {
            text: "最近三个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit("pick", [start, end]);
            }
          }
        ]
      }
    };
  },
  created() {
    if (localStorage.getItem('roleId') === '3' || localStorage.getItem('roleId') === '1') {
      this.hasPermisson = false
    }
    this.getData(1, 20);
  },
  methods: {
    onsearch(data) {
      this.getData(this.listQuery.page, this.listQuery.limit, data);
    },
    getData(page, limit, data) {
      let sendData = {
        contractStatus: "0001", // 当前页是待提交状态，写死
        sort: "id", // 根据什么字段来排序
        order: "desc", // 默认降序
        page: page, // 当前页码
        rows: limit // 每页多少数据
      };
      sendData = Object.assign(sendData, data)
      // console.log('发送', sendData)
      this.$ajax({
        method: "get",
        url: "getList",
        params: sendData
      }).then(res => {
        // console.log(res);
        this.listLoading = false;
        this.tableData = res.data.list;
        this.total = res.data.total;
      });
    },
    edit(id) {
      // console.log(id)
      this.$router.push("/edit?id=" + id);
    },
    submit(id) {
      // console.log(id);
      let sendData = {
        contractId: id,
        contractStatus: "0002"
      };
      this.$ajax({
        method: "get",
        url: "/submit",
        params: sendData
      }).then(res => {
        if (res.data.msg === "success") {
          this.$alert("提交状态", "成功", {
            confirmButtonText: "确定",
            callback: action => {
              // window.location.reload();
              this.$router.push("/");
            }
          });
        } else {
          this.$alert("提交状态", "失败", {
            confirmButtonText: "确定"
          });
        }
      });
    },
    getList(info) {
      this.getData(info.page, info.limit);
    },
    resetDateFilter() {
      this.$refs.filterTable.clearFilter("date");
    },
    clearFilter() {
      this.$refs.filterTable.clearFilter();
    },
    formatter(row, column) {
      let obj = contractMapObj
      return obj[row.contractType]
    },
    filterTag(value, row) {
      return row.tag === value;
    },
    filterHandler(value, row, column) {
      const property = column["property"];
      return row[property] === value;
    }
  },
  components: {
    Pagination,
    dSearch
  }
};
</script>