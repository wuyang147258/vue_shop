<template>
  <div>
      <!-- 面包屑组件 -->
  <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>分类参数</el-breadcrumb-item>
  </el-breadcrumb>
  <!-- 卡片视图 -->
  <el-card>
    <!-- 警告区域 -->
    <el-alert
    title="注意：只允许为第三级分类设置相关的参数"
    type="warning" :closable="false" show-icon>
  </el-alert>
  <!-- 选择商品分类区域 -->
  <el-row class="cat_opt">
    <el-col>
      <span>选择商品分类：</span>
        <el-cascader
    v-model="selectedCateKeys"
    :options="catelist"
    :props="cateProps"
    @change="handleChange"></el-cascader>
    </el-col>
  </el-row>
  <!-- tab 页签区域 -->
    <el-tabs v-model="activeName" @tab-click="handleClick">
      <!-- 添加动态参数的面板 -->
    <el-tab-pane label="动态参数" name="many">
      <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisable=true">添加参数</el-button>
      <!-- 动态参数表格 -->
      <el-table :data="manyTableData" border stripe>
        <!-- 展开行 -->
         <el-table-column type="expand">
           <template slot-scope="scope">
             <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i"  @close="handleClose(i,scope.row)" closable>{{item}}</el-tag>
            <!-- 新键文本框 -->
            <el-input
              class="input-new-tag"
              v-if="scope.row.inputVisible"
              v-model="scope.row.inputValue"
              ref="saveTagInput"
              size="small"
              @keyup.enter.native="handleInputConfirm(scope.row)"
              @blur="handleInputConfirm(scope.row)">
                </el-input>
          <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
           </template>
         </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
        <el-table-column label="操作" >
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-tab-pane>
    <!-- 添加静态属性的面板 -->
    <el-tab-pane label="静态属性" name="only">
      <el-button type="primary" size="mini"  :disabled="isBtnDisabled" @click="addDialogVisable=true">添加属性</el-button>
      <!-- 静态属性表格 -->
       <el-table :data="onlyTableData" border stripe>
        <!-- 展开行 -->
         <el-table-column type="expand">
           <template slot-scope="scope">
             <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i"  @close="handleClose(i,scope.row)" closable>{{item}}</el-tag>
            <!-- 新键文本框 -->
            <el-input
              class="input-new-tag"
              v-if="scope.row.inputVisible"
              v-model="scope.row.inputValue"
              ref="saveTagInput"
              size="small"
              @keyup.enter.native="handleInputConfirm(scope.row)"
              @blur="handleInputConfirm(scope.row)">
                </el-input>
          <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
           </template>
         </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
        <el-table-column label="操作" >
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)" >删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-tab-pane>
  </el-tabs>

  </el-card>
  <!-- 添加参数的对话框 -->
  <el-dialog
  :title="'添加'+titleText"
  :visible.sync="addDialogVisable"
  width="50%"
  @close="addDialogClosed"
  >
  <!-- 添加参数表单 -->
<el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" class="demo-ruleForm">
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="addForm.attr_name"></el-input>
  </el-form-item>
</el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisable = false">取 消</el-button>
    <el-button type="primary" @click="addParams">确 定</el-button>
  </span>
</el-dialog> 
<!-- 修改参数的对话框 -->
 <el-dialog
  :title="'修改'+titleText"
  :visible.sync="editDialogVisable"
  width="50%"
  @close="editDialogClosed"
  >
  <!-- 添加参数表单 -->
<el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-ruleForm">
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="editForm.attr_name"></el-input>
  </el-form-item>
</el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisable = false">取 消</el-button>
    <el-button type="primary" @click="editParams">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>

<script>
export default {
  created(){
      this.getCateList()
  },
  data(){
    return{
      catelist:[],
      cateProps:{
        expandTrigger: 'hover',
        value:'cat_id',
        label:'cat_name',
        children:'children'
      },
      selectedCateKeys:[],
      //被激活的页面名称
      activeName:'many',
      //动态参数的数据
      manyTableData:[],
      //静态属性的数据
      onlyTableData:[],
      //控制添加对话框
      addDialogVisable:false,
      addForm:{
        attr_name:''
      },
      //添加表单的验证规则对象
      addFormRules:{
        attr_name:[
           { required: true, message: '请输入参数名称', trigger: 'blur' },
        ]
      },
      //控制编辑对话框的显示
      editDialogVisable:false,
      //修改的表单数据对象
      editForm:{},
      //修改表单的验证规则
      editFormRules:{
        attr_name:[
          { required: true, message: '请输入参数名称', trigger: 'blur' },
        ]
      },
      newattr_vals:[]
    }
  },
  computed:{
    //如果按钮需要被禁用返回true
    isBtnDisabled(){
      if(this.selectedCateKeys.length!==3){
        return true
      }else{
        return false
      } 
    },
    cateId(){
      if(this.selectedCateKeys.length===3){
        return this.selectedCateKeys[2]
      }
    },
    //动态计算标题
    titleText(){
      if(this.activeName==='many'){
        return '动态参数'
      }
      return '静态参数'
    }
  },
  methods:{
    //获取所有的商品分类列表
    async getCateList(){
      const {data:res}=await this.$http.get('categories')
      if(res.meta.status!==200) return this.$message.error('获取商品列表失败')
      this.catelist=res.data
      
    },
    //联结选择框的变化
     handleChange(){
     this.getParamsData()
    },
    //tab页签点击事件的处理函数
    handleClick(){
        this.getParamsData()
      console.log(this.activeName);
    },
    //获取参数列表数据
   async getParamsData(){
  //证明选中的不是三级分类
      if(this.selectedCateKeys.length!==3){
        this.selectedCateKeys=[]
        this.manyTableData=[]
        this.onlyTableData=[]
        return
      }
      //选中的是三级分类
      const {data:res}=await this.$http.get(`categories/${this.cateId}/attributes`,
      {params:{
        sel:this.activeName
      }})
      if(res.meta.status!==200){
        return this.$message.error('获取参数列表失败')
      }
      //将attr_vals中字符串转为数组
      res.data.forEach(item=>{
        if(this.activeName==='many'){
          item.attr_vals=item.attr_vals ? item.attr_vals.split(',') :[]
        }else{
          item.attr_vals=item.attr_vals ? item.attr_vals.split(' ') :[]
        }
        
        //控制文本框的显示与隐藏
        item.inputVisible=false
        //文本框输入的值
        item.inputValue=''
      })
      console.log(res.data);
     if(this.activeName==='many'){
       this.manyTableData=res.data
     }else{
       this.onlyTableData=res.data
     }
    },
    //添加对话框的关闭事件
    addDialogClosed(){
      this.$refs.addFormRef.resetFields()
    },
    addParams(){
      this.$refs.addFormRef.validate(async vaild=>{
        if(!vaild)  return
       const {data:res}= await this.$http.post(`categories/${this.cateId}/attributes`,{
          attr_name:this.addForm.attr_name,
          attr_sel:this.activeName
        })
        if(res.meta.status!==201){
          return this.$message.error('添加参数失败')
        }
        this.$message.success('添加参数成功')
        this.addDialogVisable=false
        this.getParamsData()
      })
    },
    /* 显示编辑对话框*/
  async showEditDialog(id){
     this.editDialogVisable=true
    const {data:res}=await this.$http.get(`categories/${this.cateId}/attributes/${id}`,{
       params:{ attr_sel:this.activeName}
     })
     if(res.meta.status!==200) return this.$message.error('获取参数失败')
     this.editForm=res.data
   },
   //重置表单
   editDialogClosed(){
     this.$refs.editFormRef.resetFields()
   },
   //点击按钮修改参数
   editParams(){
     this.$refs.editFormRef.validate(async vaild=>{
       if(!vaild) return
       const{data:res}= await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,{
         attr_name:this.editForm.attr_name,
         attr_sel:this.activeName
       })
       if(res.meta.status!==200) return this.$message.error('修改参数失败')
       this.$message.success('修改参数成功')
       this.getParamsData()
       this.editDialogVisable=false
     })
   },
   //根据id删除
  async removeParams(id){
     const confirmResult= await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        if(confirmResult!=='confirm') return this.$message.info('已取消删除')
        //请求删除
         const {data:res} =await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
        if(res.meta.status!==200){
          return this.$message.error('删除参数失败')
        }
        this.$message.success('删除参数成功')
        this.getParamsData()
  },
  //文本失去焦点，或者抬起enter键
 async handleInputConfirm(row){
    if(row.inputValue.trim().length===0){
      row.inputValue=''
      row.inputVisible=false
      return
    }
    //如果没有return 就证明输入有效的数据
    row.attr_vals.push(row.inputValue.trim())
    row.inputValue=''
    row.inputVisible=false
    this.saveAttrVals(row)
   
  },
  async saveAttrVals(row){
    if(this.activeName==='many'){
      this.newattr_vals=row.attr_vals.join(',')
    }else{
      this.newattr_vals=row.attr_vals.join(' ')
    }
    //发起请求，保存输入数据
     const {data:res} =await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,{
      attr_name:row.attr_name,
      attr_sel:row.attr_sel,
      //将数组转成字符串，用逗号拼接
      attr_vals:this.newattr_vals
    })
    if(res.meta.status!==200){
      return this.message.error('修改参数失败')
    }
    this.$message.success('修改参数成功!')
    console.log(res.data);
  },
  //点击按钮展示文本框
  showInput(row){
    row.inputVisible=true
    //让文本框自动获取焦点
    /* $nextTick方法的作用，就是当页面上元素被重新渲染之后，才会指定回调函数中的代码 */
     this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
  },
  handleClose(i,row){
    row.attr_vals.splice(i,1)
    this.saveAttrVals(row)

  }
  }

}
</script>

<style lang="less" sscoped>
.cat_opt{
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag{
  width: 120px!important;
}
</style>