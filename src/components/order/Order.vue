<template>
  <div>
      <!-- 面包屑组件 -->
  <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>用户管理</el-breadcrumb-item>
    <el-breadcrumb-item>用户列表</el-breadcrumb-item>
  </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <!-- 头部搜索框 -->
      <el-input placeholder="请输入内容" v-model="queryInfo.query" class="input" >
    <el-button slot="append" icon="el-icon-search" @click="getOrderList"></el-button>
  </el-input>
  <!-- Table区域 -->
  <el-table :data="orderList" border stripe>
    <el-table-column type="index" label="#"></el-table-column>
       <el-table-column label="订单编号" prop="order_number" width="450px"></el-table-column>
        <el-table-column label="订单价格" prop="order_price" width="128px"></el-table-column>
         <el-table-column label="是否付款" width="130px">
           <template slot-scope="scope">
             <el-tag type="danger" v-if="scope.row.order_pay==='0'">未付款</el-tag>
             <el-tag type="success" v-else>已付款</el-tag>
           </template>
         </el-table-column>
          <el-table-column label="是否发货" prop="is_send" width="130px"></el-table-column>
           <el-table-column label="下单时间" prop="create_time" width="150px">
             <template slot-scope="scope">{{scope.row.create_time|dateFormat}}</template>
           </el-table-column>
            <el-table-column label="操作"   width="200px">
               <template >
             <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog"></el-button>
             <el-button type="success" icon="el-icon-position" size="mini" @click="showProgressBox"></el-button>
           </template>
            </el-table-column>
  </el-table>
  <!-- 分页区域 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
    <!-- 编辑对话框 -->
    
<el-dialog
  title="修改地址"
  :visible.sync="editOrderVisable"
  width="50%"
  @close="editDialogClosed"
 >
  <!-- 表单区域 -->
  <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" >
  <el-form-item label="省市区/县" prop="address1">
    <el-cascader :options="cityData" v-model="editForm.address1"></el-cascader>
  </el-form-item>
  <el-form-item label="详细地址" prop="address2">
    <el-input v-model="editForm.address2"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editOrderVisable = false">取 消</el-button>
    <el-button type="primary" @click="editOrderVisable = false">确 定</el-button>
  </span>
</el-dialog>
<!-- 物流进度对话框 -->
<el-dialog
  title="物流进度"
  :visible.sync="progressVisable"
  width="50%"
  >
  <!-- 时间线 -->
  <el-timeline>
    <el-timeline-item
      v-for="(activity, index) in progressInfo"
      :key="index"
      :timestamp="activity.time">
      {{activity.context}}
    </el-timeline-item>
  </el-timeline>
</el-dialog>
  </div>
</template>

<script>
import cityData from './citydata'
import OderList from  './wuliulList'
export default {
  created(){
    this.getOrderList()
  },
  data(){
    return{
      //查询参数
      queryInfo:{
        query:'',
        pagenum:1,
        pagesize:10           
      },
      //总共的数据条数
      total:0,
      //数据列表
      orderList:[],
      editOrderVisable:false,
      //表单数据
      editForm:{
        address1:[],
        address2:''
      },
      //表单验证规则
      editFormRules:{
        address1:[
            { required: true, message: '请选择地址', trigger: 'blur' },
        ],
         address2:[
            { required: true, message: '请填写详细地址', trigger: 'blur' },
        ]
      },
      cityData,
      progressVisable:false,
      //物流信息
      progressInfo:[]
    }
  },
  methods:{
   async getOrderList(){
      //获取order列表的数据
    const {data:res} =await this.$http.get('orders',{
        params:this.queryInfo
      })
      if(res.meta.status!==200) return this.$message.error('获取数据列表出错')
      this.$message.success('获取数据列表成功')
      this.orderList=res.data.goods
      this.total=res.data.total
    },
    //页码大小改变
    handleSizeChange(newSize){
      this.queryInfo.pagesize=newSize
      this.getOrderList()
    },
    //页码数量改变
    handleCurrentChange(newPage){
      this.queryInfo.pagenum=newPage
      this.getOrderList()
    },
    showEditDialog(){
      this.editOrderVisable=true
    },
    editDialogClosed(){
      this.$refs.editFormRef.resetFields()
    },
    //接口有问题
 /*  async showProgressBox(){
     const {data:res}= await this.$http.get('/kuaidi/1106975712662')
     if(res.meta.status!==200) return this.$message.error('获取物理信息失败')
     this.progressInfo=res.data
      this.progressVisable=true
      console.log(this.progressInfo);
    } */
    showProgressBox(){
      this.progressInfo=OderList.data
      console.log(this.progressInfo);
      this.progressVisable=true
    }
  }
}
</script>

<style lang="less" scoped>
.input{
  width: 300px!important;
}
</style>