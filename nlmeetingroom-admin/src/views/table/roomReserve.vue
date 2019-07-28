<template>
<div>
  <br>
  <el-form :inline="true">
          <el-form-item label="申请人id">
<el-input v-model="searchMap.userid" placeholder="申请人"></el-input></el-form-item>
          <el-form-item label="会议室id">
<el-input v-model="searchMap.roomid" placeholder="会议室"></el-input></el-form-item>
          <el-form-item label="开始时间">
<el-input v-model="searchMap.startdate" placeholder="开始时间"></el-input></el-form-item>
          <el-form-item label="结束时间">
<el-input v-model="searchMap.enddate" placeholder="结束时间"></el-input></el-form-item>
          <el-form-item label="内容">
<el-input v-model="searchMap.content" placeholder="内容"></el-input></el-form-item>
          <el-form-item label="0:未审核 1：通过审核">
<el-input v-model="searchMap.state" placeholder="0:未审核 1：通过审核"></el-input></el-form-item>

    <el-button type="primary" @click="fetchData()">查询</el-button>
    <el-button type="primary" @click="handleEdit('')">新增</el-button>
  </el-form>
  <el-table
    :data="list"
    border
    style="width: 100%">
          <el-table-column prop="id" label="" width="80"></el-table-column>
          <el-table-column prop="user.nickname" label="申请人" width="80"></el-table-column>
          <el-table-column prop="room.name" label="会议室" width="80"></el-table-column>
          <el-table-column prop="startdate" label="开始时间" width="80"></el-table-column>
          <el-table-column prop="enddate" label="结束时间" width="80"></el-table-column>
          <el-table-column prop="content" label="内容" width="80"></el-table-column>
          <el-table-column prop="state" label="0:未审核 1：通过审核" width="80"></el-table-column>

    <el-table-column
      fixed="right"
      label="操作"
      width="100">
      <template slot-scope="scope">
        <el-button @click="handleEdit(scope.row.id)" type="text" size="small">修改</el-button>
        <el-button @click="handleDelete(scope.row.id)" type="text" size="small">删除</el-button>
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
      :total="total">
    </el-pagination>  
  <el-dialog title="编辑" :visible.sync="dialogFormVisible">
    <el-form label-width="80px">
        <el-form-item label="申请人"><el-input v-model="pojo.userid"></el-input></el-form-item>
        <el-form-item label="会议室"><el-input v-model="pojo.roomid"></el-input></el-form-item>
        <el-form-item label="开始时间"><el-input v-model="pojo.startdate"></el-input></el-form-item>
        <el-form-item label="结束时间"><el-input v-model="pojo.enddate"></el-input></el-form-item>
        <el-form-item label="内容"><el-input v-model="pojo.content"></el-input></el-form-item>
        <el-form-item label="0:未审核 1：通过审核"><el-input v-model="pojo.state"></el-input></el-form-item>

        <el-button type="primary" @click="handleSave()">保存</el-button>
        <el-button @click="dialogFormVisible = false" >关闭</el-button>
    </el-form>
  </el-dialog>
</div>
</template>
<script>
import roomReserveApi from '@/api/roomReserve'
export default {
  data() {
    return {
      list: [],
      total: 0, // 总记录数
      currentPage: 1, // 当前页
      pageSize: 10, // 每页大小
      searchMap: {}, // 查询条件
      dialogFormVisible: false, // 编辑窗口是否可见
      pojo: {}, // 编辑表单绑定的实体对象
      cityList: [], // 城市列表
      id: '' // 当前用户修改的ID
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchDataSize(val){
        this.pageSize=val
        this.fetchData()
    },
    fetchDataCurrent(val){
        this.currentPage=val
        this.fetchData()
    },
    fetchData() {
      roomReserveApi.search(this.currentPage, this.pageSize, this.searchMap).then(response => {
        this.list = response.data.rows
        this.total = response.data.total
      })
    },
    handleSave() {
      roomReserveApi.update(this.id, this.pojo).then(response => {
        this.$message({
          message: response.message,
          type: (response.flag ? 'success' : 'error')
        })
        if (response.flag) { // 如果成功
          this.fetchData() // 刷新列表
        }
      })
      this.dialogFormVisible = false // 关闭窗口
    },
    handleEdit(id) {
      this.id = id
      this.dialogFormVisible = true // 打开窗口
      if (id !== '') { // 修改
        roomReserveApi.findById(id).then(response => {
          if (response.flag) {
            this.pojo = response.data
          }
        })
      } else {
        this.pojo = {} // 清空数据
      }
    },
    handleDelete(id) {
      this.$confirm('确定要删除此纪录吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        roomReserveApi.deleteById(id).then(response => {
          this.$message({ message: response.message, type: (response.flag ? 'success' : 'error') })
          if (response.flag) {
            this.fetchData() // 刷新数据
          }
        })
      })
    }
  }
}
</script>
