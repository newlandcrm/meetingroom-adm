<template>
<div style="margin:0px 0px 0px 30px">
  <br>
  <el-form :inline="true">
          <el-form-item  label="楼名">
<el-input v-model="searchMap.name" placeholder="楼名"></el-input></el-form-item>


    <el-button type="primary" @click="fetchData()">查询</el-button>
    <el-button type="primary" @click="handleEdit('')">新增</el-button>
  </el-form>
  <el-table
   v-loading="loading"
    :data="list"
    border
    style="width: 100%">
          <el-table-column prop="name" label="楼名" width="80"></el-table-column>
          <el-table-column prop="sort" label="权重" width="80"></el-table-column>

    <el-table-column
      
      label="操作"
      width="100">
      <template slot-scope="scope">
        <el-button @click="handleEdit(scope.row.id)" type="text" >修改</el-button>
        <el-button @click="handleDelete(scope.row.id)" type="text" >删除</el-button>
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
        <el-form-item label="楼名"><el-input v-model="pojo.name"></el-input></el-form-item>

        <el-button type="primary" @click="handleSave()">保存</el-button>
        <el-button @click="dialogFormVisible = false" >关闭</el-button>
    </el-form>
  </el-dialog>
</div>
</template>
<script>
import buildingApi from '@/api/building'
export default {
  data() {
    return {
      list: [],
      total: 0, // 总记录数
      currentPage: 1, // 当前页
      pageSize: 5, // 每页大小
      searchMap: {}, // 查询条件
      dialogFormVisible: false, // 编辑窗口是否可见
      pojo: {}, // 编辑表单绑定的实体对象
      cityList: [], // 城市列表
      id: '', // 当前用户修改的ID
      loading:true,
      value1:[],
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    test(){
      var searchMap1={"starttime":this.value1}
      buildingApi.test(searchMap1).then(response => {

      })
    },
    fetchDataSize(val){
        this.pageSize=val
        this.fetchData()
    },
    fetchDataCurrent(val){
        this.currentPage=val
        this.fetchData()
    },
    fetchData() {
       console.log(this.pageSize+"currenpage"+this.currentPage)
      buildingApi.search(this.currentPage, this.pageSize, this.searchMap).then(response => {
       
        this.list = response.data.rows
        this.total = response.data.total
         this.loading =false
      })
    },
    handleSave() {
      buildingApi.update(this.id, this.pojo).then(response => {
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
        buildingApi.findById(id).then(response => {
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
        buildingApi.deleteById(id).then(response => {
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
