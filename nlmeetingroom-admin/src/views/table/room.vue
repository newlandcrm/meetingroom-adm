<template>
<div style="margin:0px 0px 0px 30px">
  <br>
  <el-form :inline="true">
          <el-form-item label="会议室名称">
<el-input v-model="searchMap.name" placeholder="会议室名称"></el-input></el-form-item>
<el-form-item label="所属楼层">
<el-cascader
      :props="defaultParams"
      :options="options"
      @change="cascadeChage"
      :clearable="true"
  ></el-cascader>
   </el-form-item>

    <el-button type="primary" @click="fetchData()">查询</el-button>
    <el-button type="primary" @click="handleEdit('')">新增</el-button>
  </el-form>
  <el-table
    v-loading="loading"
    :data="list"
    border
    style="width: 100%">
          <el-table-column prop="name" label="会议室名称" width="80"></el-table-column>
          <el-table-column prop="floor.floors" label="所属楼层" width="80"></el-table-column>
          <el-table-column prop="floor.building.name" label="所属楼" width="80"></el-table-column>
          <el-table-column prop="capacity" label="会议室容量" width="80"></el-table-column>
          <el-table-column prop="addr" label="会议室地址" width="80"></el-table-column>
          <el-table-column prop="openstate" label="状态" width="80">
                <template slot-scope="scope">
                    <el-tag type="success"  effect="dark" v-if="scope.row.openstate==1">开放</el-tag>
                    <el-tag type="danger" effect="dark"  v-else>关闭</el-tag>
                </template>
          </el-table-column>
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
        <el-form-item label="会议室名称"><el-input v-model="pojo.name"></el-input></el-form-item>
        <el-form-item label="所属楼层">
         <el-cascader
              v-model="dialogtemp"
              :props="defaultParams"
              :options="options"
              @change="cascadeChage2"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="会议室容量"><el-input v-model="pojo.capacity"></el-input></el-form-item>
        <el-form-item label="会议室地址"><el-input v-model="pojo.addr"></el-input></el-form-item>
        <el-form-item label="开放状态">
           <el-switch
            v-model="pojo.openstate"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :active-value="1"
            :inactive-value="0">
          </el-switch>
        </el-form-item>
        <el-button type="primary" @click="handleSave()">保存</el-button>
        <el-button @click="dialogFormVisible = false" >关闭</el-button>
    </el-form>
  </el-dialog>
</div>
</template>
<script>
import roomApi from '@/api/room'
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
      defaultParams: {
         label: 'name',
         value: 'id',
         children: 'floors'
     },
      options: [],
      dialogtemp:[],
      loading:true
    }
  },
  created() {
    this.fetchData()
     buildingApi.getList().then(response => {
        this.options=response.data
        for ( var i = 0; i <this.options.length; i++){
            // if(this.options[i].floors.length<1){
            //   this.options[i].floors=undefined
            // }
            for ( var j = 0; j <this.options[i].floors.length; j++){
                this.options[i].floors[j].name=this.options[i].floors[j].floors
            }  
        }     
        
     })
  },
  methods: {
    cascadeChage(value){
      if(value.length!=0){
        this.searchMap.floorid=value[value.length-1]
        console.log(value[value.length-1]+" "+this.searchMap.floorid)
      }else{
        this.searchMap.floorid=''
      }
      
    },
    cascadeChage2(value){
      if(value.length!=0){
        this.pojo.floorid=value[value.length-1]
        console.log(value[value.length-1]+" "+value);
      }else{
        this.pojo.floorid=''
      }
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
      roomApi.search(this.currentPage, this.pageSize, this.searchMap).then(response => {
        this.list = response.data.rows
        this.total = response.data.total
        this.loading =false
      })
    },
    handleSave() {
  
      roomApi.update(this.id, this.pojo).then(response => {
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
      this.dialogtemp =[]
      this.id = id
      this.dialogFormVisible = true // 打开窗口
      
      if (id !== '') { // 修改
        roomApi.findById(id).then(response => {
          if (response.flag) {
            this.pojo = response.data
            // this.pojo.openstate = response.data.openstate
            this.dialogtemp.push(this.pojo.floor.building.id)
            this.dialogtemp.push(this.pojo.floor.id)
            
          }
        })
      } else {
        this.pojo.name ='' // 清空数据
        this.pojo.floorid =''
        this.pojo.capacity =''
        this.pojo.addr =''
      }
    },
    handleDelete(id) {
      this.$confirm('确定要删除此纪录吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        roomApi.deleteById(id).then(response => {
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
