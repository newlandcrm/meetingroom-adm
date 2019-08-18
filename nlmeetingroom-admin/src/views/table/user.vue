<template>
<div style="margin:0px 0px 0px 15px">
  <br>
  <el-form :inline="true">
          <el-form-item label="账号">
<el-input v-model="searchMap.username" placeholder="账号"></el-input></el-form-item>
          <el-form-item label="昵称">
<el-input v-model="searchMap.nickname" placeholder="昵称"></el-input></el-form-item>
<el-form-item label="所属部门">
  <el-select v-model="searchMap.departmentid" placeholder="请选择">
    <el-option
      v-for="item in options"
      :key="item.id"
      :label="item.name"
      :value="item.id">
    </el-option>
  </el-select>
 </el-form-item>
    <el-button type="primary" @click="fetchData()">查询</el-button>
  </el-form>
  <el-table
   v-loading="loading"
    :data="list"
    border
    style="width: 100%">
          <el-table-column prop="username" label="账号" ></el-table-column>
          <el-table-column prop="mobile" label="手机号码"></el-table-column>
          <el-table-column prop="nickname" label="昵称"></el-table-column>
          <el-table-column prop="email" label="E-Mail"></el-table-column>
          <el-table-column prop="regdate" label="注册日期" >
             <template slot-scope="scope">{{ scope.row.regdate | formatDate}}</template> 
          </el-table-column>
          <el-table-column prop="lastdate" label="最后登陆日期">
             <template slot-scope="scope">{{ scope.row.lastdate | formatDate}}</template> 
          </el-table-column>
          <el-table-column prop="roleid=='1'?'管理员':'用户'" label="角色  ">
            <template slot-scope="scope">
                <span v-if="scope.row.roleid==='1'" >管理员</span>
                <span v-else >用户</span>
            </template>
          </el-table-column>
          <el-table-column prop="department.name" label="所属部门"></el-table-column>

    <el-table-column
      label="操作"
     >
      <template slot-scope="scope">
        <el-button @click="handleEdit(scope.row.id)" type="text" >修改</el-button>
        <!-- <el-button @click="handleDelete(scope.row.id)" type="text" >删除</el-button> -->
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
        <el-form-item label="手机号码"><el-input v-model="pojo.mobile"></el-input></el-form-item>
        <el-form-item label="昵称"><el-input v-model="pojo.nickname"></el-input></el-form-item>
        <el-form-item label="E-Mail"><el-input v-model="pojo.email"></el-input></el-form-item>
        <el-form-item label="角色："><el-input v-model="pojo.roleid"></el-input></el-form-item>
        <el-form-item label="所属部门">
            <el-select v-model="pojo.department.id" placeholder="请选择">
              <el-option
                v-for="item in addOptions"
                :key="item.id"
                :label="item.name"
                :value="item.id">
              </el-option>
            </el-select>
        </el-form-item>
        <el-button type="primary" @click="handleSave()">保存</el-button>
        <el-button @click="dialogFormVisible = false" >关闭</el-button>
    </el-form>
  </el-dialog>
</div>
</template>
<script>
import userApi from '@/api/user'
import { formatTime } from "@/utils/index";
import departmentApi from '@/api/department'
export default {
  data() {
    return {
      list: [],
      total: 0, // 总记录数
      currentPage: 1, // 当前页
      pageSize: 5, // 每页大小
      searchMap: {}, // 查询条件
      dialogFormVisible: false, // 编辑窗口是否可见
      pojo: {department:{id:''},}, // 编辑表单绑定的实体对象
      cityList: [], // 城市列表
      id: '', // 当前用户修改的ID
      loading:true,
      options: [{
          id: '',
          name: '全部'
      }],
      addOptions: [],
    }
  },
  created() {
    this.fetchData()
     departmentApi.getList().then(response => {
       this.addOptions=response.data
       this.options=this.options.concat(response.data)
      })
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
      userApi.search(this.currentPage, this.pageSize, this.searchMap).then(response => {
        this.list = response.data.rows
        this.total = response.data.total
         this.loading =false
      })
    },
    handleSave() {
      userApi.update(this.id, this.pojo).then(response => {
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
        userApi.findById(id).then(response => {
          if (response.flag) {
            this.pojo = response.data
          }
        })
      } else {
        this.pojo = {} // 清空数据
      }
    },
    
    // handleDelete(id) {
    //   this.$confirm('确定要删除此纪录吗?', '提示', {
    //     confirmButtonText: '确定',
    //     cancelButtonText: '取消',
    //     type: 'warning'
    //   }).then(() => {
    //     userApi.deleteById(id).then(response => {
    //       this.$message({ message: response.message, type: (response.flag ? 'success' : 'error') })
    //       if (response.flag) {
    //         this.fetchData() // 刷新数据
    //       }
    //     })
    //   })
    // }
  },
  filters: {
    formatDate(val) {
      return formatTime(val, "{y}-{m}-{d} {h}:{i}:{s}");
    }
  }
}
</script>
