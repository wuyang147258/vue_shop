<template>
  <div>
        <!-- 面包屑组件 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>商品分类</el-breadcrumb-item>
  </el-breadcrumb>
  <!-- 卡片视图区 -->
  <el-card>
    <el-row>
      <el-col><el-button type="primary" @click="showAddCateDialog">添加分类</el-button></el-col>
    </el-row>
    <!-- 表格 -->
    <tree-table class="tree-table" :data="catelist" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border>
      <template slot="isok" slot-scope="scope">
        <i class="el-icon-success" style="color:green" v-if="scope.row.cat_deleted===false"></i>
        <i class="el-icon-error" style="color:red" v-else></i>
      </template>
      <!-- 排序 -->
      <template slot="order" slot-scope="scope">
        <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
         <el-tag size="mini" type="success" v-else-if="scope.row.cat_level===1">二级</el-tag>
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
      </template>
      <!-- 操作 -->
      <template slot="opt" slot-scope="scope">
        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showCateEditDialog(scope.row.cat_id)">编辑</el-button>
        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCate(scope.row.cat_id)">删除</el-button>
      </template>
    </tree-table>
    <!-- 分页 -->
     <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[3, 5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </el-card>
  <!-- 这是添加分类对话框 -->
  <el-dialog
  title="添加分类"
  :visible.sync="addCateDIalogVisable"
  width="50%"
  @close="addCateDialogClosed"
  >
  <!-- 添加分类的表单 -->
  <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
  <el-form-item label="分类名称" prop="cat_name">
    <el-input v-model="addCateForm.cat_name"></el-input>
  </el-form-item>
   <el-form-item label="父级分类">
     <!-- options用来指定数据源 props用来指定配置对象-->
      <el-cascader
    v-model="selectedKeys"
    :options="parentCateList"
    :props="caseaderProps"
    @change="parentCateChanged" clearable></el-cascader>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDIalogVisable = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
</el-dialog>
<!-- 这是编辑的对话框 -->

<el-dialog
  title="编辑分类"
  :visible.sync="editCateVisable"
  width="50%"
 >
  <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="100px" class="demo-ruleForm">
  <el-form-item label="分类名称" prop="cat_name">
    <el-input v-model="editCateForm.cat_name"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editCateVisable = false">取 消</el-button>
    <el-button type="primary" @click="saveEditCate">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>

<script>
import RightsVue from '../power/Rights.vue'
export default {
  created(){
    this.getCateList()
  },
    data(){
      return{
        //查询条件
        queryInfo:{
          type:3,
          pagenum:1,
          pagesize:5
        },
        //商品分类的数据列表
        catelist:[],
        //总数据条数
        total:0,
        columns:[{
          label:'分类名称',
          prop:'cat_name'
        },{
          label:'是否有效',
          type:'template',
          template:'isok'
        },{
          label:'排序',
           type:'template',
           template:'order'
        },{
          label:'操作',
          type:'template',
           template:'opt'
        }],
        //控制分类对话框的显示
        addCateDIalogVisable:false,
        //添加分类的表单对象
        addCateForm:{
          cat_name:'',
          //父级分类id
          cat_pid:0,
          //分类的等级，默认是一级
          cat_level:0
        },
        //添加分类表单的验证规则
        addCateFormRules:{
          cat_name:[
             { required: true, message: '请输入分类名称', trigger: 'blur' },
          ]
        },
        //父级分类的列表
        parentCateList:[],
        //指定联级选择器的配置对象
        caseaderProps:{
          expandTrigger: 'hover',
          checkStrictly: true,
          value:'cat_id',
          label:'cat_name',
          children:'children'
        },
        //选中的父级分类的数组
        selectedKeys:[],
        //控制编辑对话框的显示
        editCateVisable:false,
        //存储查询到的分类信息
        editCateForm:{},
         //编辑分类的表单验证规则
        editCateFormRules:{
          cat_name:[
              { required: true, message: '请输入分类名称', trigger: 'blur' },
          ]
        },
  
      }
     
      
    },
    methods:{
      //获取商品分类的数据
    async  getCateList(){
    const  {data:res}= await this.$http.get('categories',{
          params:this.queryInfo
        })
       
        if(res.meta.status!==200) return this.$message.error('获取商品列表失败')
        this.catelist=res.data.result
        this.total=res.data.total
      },
      //监听pagesize 改变
      handleSizeChange(newSize){
        this.queryInfo.pagesize=newSize
        this.getCateList()
      },
      //监听pagenum 改变
      handleCurrentChange(newPage){
        this.queryInfo.pagenum=newPage
        this.getCateList()
      },
      //点击按钮显示对话框
      showAddCateDialog(){
        this.getParentCateList()
        this.addCateDIalogVisable=true
        
      },
      //获取父级分类的数据列表
     async getParentCateList(){
      const {data:res}=await this.$http.get('categories',{
          params:{type:2}
        })
        if(res.meta.status!==200) return this.$message.error('获取父类数据失败')
        this.parentCateList=res.data
      },
      //选择项发生变化
      parentCateChanged(){
        console.log(this.selectedKeys);
        //如果selectedKeys数组中的length大于0，证明选中的父级分类
        if(this.selectedKeys.length>0){
          this.addCateForm.cat_pid=this.selectedKeys[this.selectedKeys.length-1]
      
        //为当前分类的等级赋值
        this.addCateForm.cat_level=this.selectedKeys.length
        return  }
        else{
          this.addCateForm.cat_pid=0
          this.addCateForm.cat_level=0
        }
      },
      addCate(){
        this.$refs.addCateFormRef.validate(async valid=>{
          if(!valid) return
        const {data:res}=await this.$http.post('categories',this.addCateForm)
        console.log(res.data);
        if(res.meta.status!==201) return this.$message.error('添加分类失败')
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDIalogVisable=false
        })
      },
      addCateDialogClosed(){
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys=[]
        this.addCateForm.cat_pid=0
         this.addCateForm.cat_level=0
      },
      async showCateEditDialog(id){
        this.editCateVisable=true
        //通过id查询分类信息
        const {data:res} =await this.$http.get('categories/'+id)
        if(res.meta.status!==200) return this.$message.error('获取分类信息错误')
        this.editCateForm=res.data
        console.log(this.editCateForm.cat_id);
      },
      saveEditCate(){
        //进行预验证
      this.$refs.editCateFormRef.validate(async valid=>{
      
        if(!valid) return
         const {data:res}=await this.$http.put('categories/'+this.editCateForm.cat_id,{
           cat_name:this.editCateForm.cat_name
         })
          if(res.meta.status!==200) return this.$message.error('更改分类出错')
          this.$message.success('修改分类成功')
          this.getCateList()
          this.editCateVisable=false
      })
      },
      async removeCate(id){
    //弹窗是否删除
    const confirmResult= await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
          //如果用户确认删除，则返回值为字符串 confirm
        if(confirmResult!=='confirm'){
          return this.$message.info('已取消删除')
        }
        //确认删除
      const {data:res}=await this.$http.delete('categories/'+id)
        if(res.meta.status!==200) return this.$message.error('删除角色失败')
        this.$message.success('成功删除分类')
        this.getCateList()
    },
    }
}
</script>


<style lang="less" scoped>

</style>