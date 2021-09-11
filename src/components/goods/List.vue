<template>
  <div>
      <!-- 面包屑组件 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>商品分类</el-breadcrumb-item>
  </el-breadcrumb>
  <!-- 卡片视图 -->
  <el-card>
    <el-row :gutter="20">
      <el-col :span="8">
         <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodList">
         
    <el-button slot="append" icon="el-icon-search" @click="getGoodList"></el-button>
  </el-input>
      </el-col>
      <el-col :span="4">
        <el-button type="primary" @click="goAddpage">添加商品</el-button>
      </el-col>
    </el-row>
    <!-- table表格区域 -->
    <el-table :data="goodsList" border stripe>
      <el-table-column type="index"></el-table-column>
      <el-table-column label="商品名称" prop="goods_name" 
      ></el-table-column>
      <el-table-column label="商品价格(元)" prop="goods_price" width="95px
      "></el-table-column>
       <el-table-column label="商品重量" prop="goods_weight" width="70px
       "></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px
        "><template slot-scope="scope">{{scope.row.add_time|dateFormat}}</template>
        </el-table-column>
         <el-table-column label="操作" width="130px
         ">
           <template slot-scope="scope">
             <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.goods_id)"></el-button>
             <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.goods_id)"></el-button>
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
      :total="total" background>
    </el-pagination>
  </el-card>
  <!-- 编辑对话框 -->
  <el-dialog
  title="编辑商品信息"
  :visible.sync="editDialogVisable"
  width="50%"
  @close="closeEditDialog"
  >
  <!-- 编辑表单区域 -->
  <el-form :model="editForm" :rules="editFormRules" ref="editRef" label-width="100px" class="demo-ruleForm">
  <el-form-item label="商品名称" prop="goods_name">
    <el-input v-model="editForm.goods_name"></el-input>
  </el-form-item>
  <el-form-item label="价格" prop="goods_price">
    <el-input v-model="editForm.goods_price "></el-input>
  </el-form-item>
  <el-form-item label="数量" prop="goods_number">
    <el-input v-model="editForm.goods_number"></el-input>
  </el-form-item>
   <el-form-item label="重量" prop="goods_weight">
    <el-input v-model="editForm.goods_weight "></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisable = false">取 消</el-button>
    <el-button type="primary" @click="saveEditForm">确 定</el-button>
  </span>
</el-dialog>

  </div>
</template>

<script>
export default {
  created(){
    this.getGoodList()
  },
  data(){
    return{
      //查询参数对象
      queryInfo:{
        query:'',
        pagenum:1,
        pagesize:10
      },
      //查询到的数据列表
      goodsList:[],
      //数据总条数
      total:0,
      editDialogVisable:false,
      editForm:{},
      //编辑的表单规则
      editFormRules:{
        goods_name:[
           { required: true, message: '请输入商品名称', trigger: 'blur' },
        ],
        goods_price:[
           { required: true, message: '请输入价格', trigger: 'blur' },
        ],
        goods_number:[
           { required: true, message: '请输入数量', trigger: 'blur' },
        ],
        goods_weight:[ 
          { required: true, message: '请输入重量', trigger: 'blur' },
        ]
      }
    }
  },
  methods:{
   async getGoodList(){
     const {data:res} =await this.$http.get('goods',{
        params:this.queryInfo
      })
      if(res.meta.status!==200) return this.$message.error('获取商品列表失败')
      this.$message.success('获取数据成功')
      this.goodsList=res.data.goods
      this.total=res.data.total
    },
    //获取最新的页数大小
    handleSizeChange(newSize){
      this.queryInfo.pagesize=newSize
      this.getGoodList()
    },
    //获取最新的页码数
    handleCurrentChange(newPage){
      this.queryInfo.pagenum=newPage
      this.getGoodList()
    },
    //删除商品
   async removeById(id){
      const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        if(confirmResult!=='confirm') return this.$message.info('已经取消删除')
        //请求删除
        const {data:res} =await this.$http.delete(`goods/${id}`)
        if(res.meta.status!==200) return this.$message.error('删除商品失败')
        this.$message.success('删除商品成功')
        this.getGoodList()
    },
    goAddpage(){
      this.$router.push('/goods/add')
    },
    //显示编辑对话框
   async showEditDialog(id){
      //显示对话框的同时根据id获取商品的信息,发起请求
     const {data:res} = await this.$http.get('goods/'+id)
     if(res.meta.status!==200)  this.$message.error('获取商品信息错误')
     this.editForm=res.data
     console.log(this.editForm);
      //显示对话框
      this.editDialogVisable=true
    },
    closeEditDialog(){
        this.$refs.editRef.resetFields()
    },
    saveEditForm(){
      
      this.$refs.editRef.validate(async valid=>{
        if(!valid) return this.$message.error('请完成所有表单')
      const {data:res}= await this.$http.put('goods/'+this.editForm.goods_id,{
           goods_name:this.editForm.goods_name,
           goods_price:this.editForm.goods_price,
           goods_number:this.editForm.goods_number,
           goods_weight:this.editForm.goods_weight,
          goods_cat:this.editForm.goods_cat
        })
      
        if(res.meta.status!==200) return this.$message.error('编辑商品信息错误')
        this.$message.success('修改商品信息成功')
        this.getGoodList()
        this.editDialogVisable=false
      })
    }
  }

}
</script>

<style lang="less" scoped>

</style>