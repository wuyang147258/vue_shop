<template>
  <div>
     <!-- 面包屑组件 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>添加商品</el-breadcrumb-item>
   </el-breadcrumb>
   <!-- 卡片视图 -->
   <el-card>
     <!-- 提示框区域 -->
       <el-alert
    title="添加商品区域"
    type="info"
    show-icon :closable="false">
  </el-alert>
  <!-- 步骤条区域 -->
  <el-steps :space="200" :active="activeIndex-0" finish-status="success" align-center>
  <el-step title="基本信息"></el-step>
  <el-step title="商品参数"></el-step>
  <el-step title="商品属性"></el-step>
  <el-step title="商品图片"></el-step>
  <el-step title="商品内容"></el-step>
  <el-step title="完成"></el-step>
</el-steps>
<!-- tab区域 -->
<el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
  <el-tabs :tab-position="'left'"  v-model="activeIndex" :before-leave="beforeTabLeave" @tab-click="tabclick">
    <el-tab-pane label="基本信息" name="0">
     <el-form-item label="商品名称" prop="goods_name">
        <el-input v-model="addForm.goods_name"></el-input>
    </el-form-item>
      <el-form-item label="商品价格" prop="goods_price">
        <el-input v-model="addForm.goods_price" type="number"></el-input>
    </el-form-item>
      <el-form-item label="商品重量" prop="goods_weight">
        <el-input v-model="addForm.goods_weight" type="number"></el-input>
    </el-form-item>
      <el-form-item label="商品数量" prop="goods_number">
        <el-input v-model="addForm.goods_number" type="number"></el-input>
    </el-form-item>
    <el-form-item label="商品分类" prop="goods_cat">
       <el-cascader
    v-model="addForm.goods_cat"
    :options="catelist"
    :props="cateProps"
    @change="handleChange"></el-cascader> </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品参数" name="1">
      <!-- 渲染表单的Item -->
      <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
         <el-checkbox-group v-model="item.attr_vals">
            <el-checkbox :label="cb" v-for="(cb,i) in item.attr_vals" :key="i" border size="mini"></el-checkbox>
         </el-checkbox-group>
      </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品属性" name="2">
      <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
        <el-input v-model="item.attr_vals"></el-input>
      </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品图片" name="3">
      <!-- 图片上传 -->
      <!-- action表示要上传到的后台api接口 -->
      <el-upload 
      action="http://127.0.0.1:8888/api/private/v1/upload"
      :on-preview="handlePreview"
      :on-remove="handleRemove"
      list-type="picture"
      :headers="headerObj"
      :on-success="handleSuccess" >
  <el-button size="small" type="primary">点击上传</el-button>
  <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
</el-upload>
    </el-tab-pane>
    <el-tab-pane label="商品内容" name="4">
      <!-- 富文本编辑器 -->
       <quill-editor v-model="addForm.goods_introduce">
       </quill-editor>
       <!-- 添加商品按钮 -->
       <el-button type="primary" class="btnAdd" @click="add">添加商品</el-button>
    </el-tab-pane>
  </el-tabs>
  </el-form>
   </el-card>
   <!-- 图片预览 -->
   <el-dialog
  title="图片预览"
  :visible.sync="previewVisable"
  width="50%"
 >
  <img :src="previewPath" alt="" class="previewImg">
  <span slot="footer" class="dialog-footer">
    
  </span>
</el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  created(){
    this.getCateList()
  },
  data(){
    return{
      activeIndex:'0',
      addForm:{
        goods_name:'',
        goods_price:0,
        goods_weight:0,
        goods_number:0,
        goods_cat:[],
          //图片的数组
        pics:[],
        //商品详情描述
        goods_introduce:'',
        attrs:[]
      },
      addFormRules:{
        goods_name:[
           { required: true, message: '请输入商品名称', trigger: 'blur' },
        ],
        goods_price:[
          { required: true, message: '请输入商品价格', trigger: 'blur' },
        ],
        goods_weight:[
          { required: true, message: '请输入商品重量', trigger: 'blur' },
        ],
        goods_number:[
           { required: true, message: '请输入商品数量', trigger: 'blur' },
        ],
        goods_cat:[
           { required: true, message: '请选择商品分类', trigger: 'blur' },
        ]
      },
      //商品分类列表
      catelist:[],
      //联级选择器配置
      cateProps:{
         expandTrigger: 'hover',
         label:'cat_name',
         value:'cat_id',
         children:'children'
      },
      //动态参数列表
      manyTableData:[],
      //静态属性列表
      onlyTableData:[],
      //图片上传组件的请求头
      headerObj:{
        Authorization:window.sessionStorage.getItem('token')
      },
      //图片url
      previewPath:'',
      previewVisable:false
    }
  },
  computed:{
    cateId(){
      if(this.addForm.goods_cat.length===3){
        return this.addForm.goods_cat[2]
      }
      return null
    }
  },
  methods:{
    //获取商品分类
   async getCateList(){
      const {data:res}= await this.$http.get('categories')
      if(res.meta.status!==200) return this.$message.error('获取商品分类失败')
      this.catelist=res.data
     
    },
    //级联选择求选中项变化
    handleChange(){
      if(this.addForm.goods_cat.length!==3){
        this.addForm.goods_cat=[]
      }
    },
    beforeTabLeave(acitveName,oldActiveName){
       if(oldActiveName==='0'&& this.addForm.goods_cat.length!==3){
         this.$message.error('请选择商品分类')
         return false
      }
    },
   async tabclick(){
      if(this.activeIndex==='1'){
      const {data:res}= await this.$http.get(`categories/${this.cateId}/attributes`,{
          params:{
            sel:'many'
          }
        })
        if(res.meta.status!==200){
          return this.$message.error('获取动态参数列表失败')
        }
        res.data.forEach(item=>{
        item.attr_vals= item.attr_vals.length===0? []:item.attr_vals.split(',')
        })
       this.manyTableData=res.data
      }else if(this.activeIndex==='2'){
           const {data:res}= await this.$http.get(`categories/${this.cateId}/attributes`,{
          params:{
            sel:'only'
          }
        })
        if(res.meta.status!==200) return this.$message.error('获取商品属性出错')
        this.onlyTableData=res.data
      }
    },
    //处理图片预览效果
    handlePreview(file){
      this.previewPath=file.response.data.url
      this.previewVisable=true
    },
    //点击移除图片
    handleRemove(file){
      //要移除图片的临时路径
      const filePath=file.response.data.tmp_path
      //找到对应图片的索引值
      const i= this.addForm.pics.findIndex(x=>x.pic===filePath)
      this.addForm.pics.splice(i,1)
     
    },
    //监听图片上传成功
    handleSuccess(res){
     const picInfo={pic:res.data.tmp_path}
     this.addForm.pics.push(picInfo)
    
    },
    //添加商品
    add(){
      this.$refs.addFormRef.validate(async valid=>{
        if(!valid){
          return this.$message.error('请完成全部表单')
        }
        //执行添加的业务
        //对数据进行处理
        const form=_.cloneDeep(this.addForm)
        form.goods_cat=form.goods_cat.join(',')
        //处理动态参数和静态属性数据
        this.manyTableData.forEach(item=>{
          const newInfo={attr_id:item.attr_id,attr_value:item.attr_vals.join(' ')}
          this.addForm.attrs.push(newInfo)
        })
         this.onlyTableData.forEach(item=>{
          const newInfo={attr_id:item.attr_id,attr_value:item.attr_vals}
          this.addForm.attrs.push(newInfo)
        })
        form.attrs=this.addForm.attrs
        console.log(form);

        //发起请求添加商品
       const {data:res}= await this.$http.post('goods',form)
       console.log(res);
       if(res.meta.status!==201) return this.$message.error('添加商品失败')
       this.$message.success('添加商品成功')
       this.$router.push('/goods')

      })
    }
  }
}
</script>

<style lang="less">
.el-cascader{
  width: 300px!important;
}
.el-checkbox{
  margin: 0 15px 0 0!important;
}
.previewImg{
  width: 100%;
}
.btnAdd{
  margin-top: 15px!important;
}

</style>