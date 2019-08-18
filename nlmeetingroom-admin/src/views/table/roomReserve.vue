<template>
  <div style="margin:0px 0px 0px 15px">
    <br />
    <el-form :inline="true">
      <!-- <el-form-item label="会议室id">
      <el-input v-model="searchMap.roomid" placeholder="会议室"></el-input></el-form-item>-->
      <!-- <el-form-item label="开始时间">
<el-input v-model="searchMap.startdate" placeholder="开始时间"></el-input></el-form-item>
          <el-form-item label="结束时间">
      <el-input v-model="searchMap.enddate" placeholder="结束时间"></el-input></el-form-item>-->
      <!-- <el-form-item label="内容">
      <el-input v-model="searchMap.content" placeholder="内容"></el-input></el-form-item>-->
      <!-- <el-form-item label="审核状态">
      <el-input v-model="searchMap.state" placeholder="0:未审核 1：通过审核"></el-input></el-form-item>-->
      <el-form-item label="审核状态">
        <el-select v-model="searchMap.state" placeholder="请选择">
          <el-option v-for="item in options" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
      </el-form-item>
      <el-button type="primary" @click="fetchData()">查询</el-button>
    </el-form>
    <el-table v-loading="loading" :data="list" border style="width: 100%">
      <el-table-column prop="user.nickname" label="申请人"></el-table-column>
      <el-table-column prop="room.name" label="会议室"></el-table-column>
      <el-table-column label="开始时间">
        <template slot-scope="scope">{{ scope.row.startdate | formatDate}}</template>
      </el-table-column>
      <el-table-column label="结束时间">
        <template slot-scope="scope">{{ scope.row.enddate | formatDate}}</template>
      </el-table-column>
      <el-table-column prop="content" label="内容"></el-table-column>
      <el-table-column prop="state" label="状态">
        <template slot-scope="scope">
          <el-tag type effect="dark" v-if="scope.row.state==0">未审核</el-tag>
          <el-tag type="success" effect="dark" v-if="scope.row.state==1">已通过</el-tag>
          <el-tag type="danger" effect="dark" v-if="scope.row.state==2">未通过</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button v-if="scope.row.state==0" @click="handleExamine(scope.row.id,1)" type="text">通过</el-button>
          <el-button v-if="scope.row.state==0" @click="handleExamine(scope.row.id,2)" type="text">驳回</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="fetchDataSize"
      @current-change="fetchDataCurrent"
      :current-page="currentPage"
      :page-sizes="[5,10,20]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>
  </div>
</template>
<script>
import roomReserveApi from "@/api/roomReserve";
import { formatTime } from "@/utils/index";
export default {
  data() {
    return {
      list: [],
      total: 0, // 总记录数
      currentPage: 1, // 当前页
      pageSize: 5, // 每页大小
      searchMap: {}, // 查询条件
      examineMap: {},
      dialogFormVisible: false, // 编辑窗口是否可见
      pojo: {}, // 编辑表单绑定的实体对象
      cityList: [], // 城市列表
      id: "", // 当前用户修改的ID
      loading: true,
      options: [
        {
          id: "",
          name: "全部"
        },
        {
          id: 0,
          name: "未审核"
        },
        {
          id: 1,
          name: "已通过"
        },
        {
          id: 2,
          name: "未通过"
        }
      ]
    };
  },
  created() {
    console.log(formatTime("1562321766000", "{y}-{m}-{d} {h}:{i}:{s}"));
    this.fetchData();
  },
  methods: {
    fetchDataSize(val) {
      this.pageSize = val;
      this.fetchData();
    },
    fetchDataCurrent(val) {
      this.currentPage = val;
      this.fetchData();
    },
    fetchData() {
      roomReserveApi
        .search(this.currentPage, this.pageSize, this.searchMap)
        .then(response => {
          this.list = response.data.rows;
          this.total = response.data.total;
          this.loading = false;
        });
    },
    handleSave() {
      roomReserveApi.update(this.id, this.pojo).then(response => {
        this.$message({
          message: response.message,
          type: response.flag ? "success" : "error"
        });
        if (response.flag) {
          // 如果成功
          this.fetchData(); // 刷新列表
        }
      });
      this.dialogFormVisible = false; // 关闭窗口
    },
    handleExamine(rsid, flag) {
      if (flag == 1) {
        this.$confirm("确定吗?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }).then(() => {
          this.examineMap.rsid = rsid;
          this.examineMap.flag = flag;
          this.examineMap.message = "";
          roomReserveApi.examine(this.examineMap).then(response => {
            this.$message({
              message: response.message,
              type: response.flag ? "success" : "error"
            });
            if (response.flag) {
              // 如果成功
              this.fetchData(); // 刷新列表
            }
          });
        });
      } else if (flag == 2) {
        this.$prompt("请输入驳回原因", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          inputPattern:  /^[\s\S]*.*[^\s][\s\S]*$/,
          inputErrorMessage: '请输入'
        }).then(({ value }) => {
          this.examineMap.rsid = rsid;
          this.examineMap.flag = flag;
          this.examineMap.message = value;
          roomReserveApi.examine(this.examineMap).then(response => {
            this.$message({
              message: response.message,
              type: response.flag ? "success" : "error"
            });
            if (response.flag) {
              // 如果成功
              this.fetchData(); // 刷新列表
            }
          });
        });
      }
    }
  },
  filters: {
    formatDate(val) {
      return formatTime(val, "{y}-{m}-{d} {h}:{i}:{s}");
    }
  }
};
</script>
