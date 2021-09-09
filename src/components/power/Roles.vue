<template>
  <div>
   <!-- 面包屑组件 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>权限管理</el-breadcrumb-item>
    <el-breadcrumb-item>角色列表</el-breadcrumb-item>
  </el-breadcrumb>
   <!-- 卡片视图 -->
   <el-card>
     <!-- 添加角色按钮区 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCharDialog">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
         <el-table-column type="expand">
           <template slot-scope="scope">
             <el-row :class="['bdbottom',i1===0 ? 'bdtop':'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id"> 
               <!-- 渲染一级权限 -->
               <el-col :span="5"><el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag><i class="el-icon-caret-right"></i></el-col>
               <!-- 渲染二级权限 -->
               <el-col :span="19">
                 <el-row :class="[i2===0?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                   <el-col :span="6"><el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag><i class="el-icon-caret-right"></i></el-col>
                   <el-col :span="18">
                     <el-tag v-for="(item3) in item2.children" :key="item3.id" type="warning" closable @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
                   </el-col>
                 </el-row>
               </el-col>
             </el-row>
           </template>
         </el-table-column>
        <!-- 索引列 -->
          <el-table-column type="index"></el-table-column>
          <el-table-column label="角色名称" prop="roleName"></el-table-column>
           <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeInfo(scope.row.id)">删除</el-button>
                <el-button  size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
              </template>
            </el-table-column>
      </el-table>
   </el-card>
   <!-- 编辑的对话框 -->
   <el-dialog
  title="编辑角色"
  :visible.sync="editDialogVisible"
  width="50%"
  @close="editDialogClosed"
 >
  <el-form ref="editFormRef" :mobel="editForm" label-width="80px
  ">
  <p>角色职务是：{{editForm.roleName}}</p>
   <el-form-item label="角色描述" >
    <el-input v-model="editForm.roleDesc" type="textarea" ></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editUserInfo">确 定</el-button>
  </span>
</el-dialog>
      <!-- 分配权限的对话框 -->
  <el-dialog
    title="分配权限"
  :visible.sync="setRightDialogVisable"
  width="30%"
  @close="setRightDialogClosed"
  >
  <!-- 树形空间 -->
  <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all	:default-checked-keys="defKeys"	 ref="treeRef"></el-tree>
  <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisable = false">取 消</el-button>
    <el-button type="primary" @click="allotRights" >确 定</el-button>
  </span>
</el-dialog>
<!-- 添加角色的对话框 -->
<el-dialog
  title="添加角色"
  :visible.sync="addDialogVisible"
  width="30%"

  >
     <!-- 内容主题区域 -->
  <el-form ref="addFormRef" :model="addForm" label-width="70px">
  <el-form-item label="用户名">
    <el-input v-model="addForm.roleName"></el-input>
  </el-form-item>
   <el-form-item label="角色描述">
    <el-input v-model="addForm.roleDesc "></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCharector">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>

<script>
export default {
    created(){
      this.getRoleList()
    },
    data(){
      return{
        //所有角色列表数据
        rolelist:[],
        //控制分配权限对话框
        setRightDialogVisable:false,
        //所有权限的数据
        rightslist:[],
        //树形控件的属性绑定对象
        treeProps:{
          label:'authName',
          children:'children'
        },
        //默认选中的id值数组
        defKeys:[],
        //即将分配权限的角色id
        roleId:'',
        //控制显示对话框
        editDialogVisible:false,
        //获取的用户信息
        editForm:{},
         //控制添加用户对话框
        addDialogVisible:false,
        addForm:{
          roleName:'',
          roleDesc :''
        }
      }   
    },
    methods:{
     async getRoleList(){
      const {data:res}=await this.$http.get('roles')
      if(res.meta.status!==200) return this.$message.error('获取列表信息错误')
      this.rolelist=res.data
      },
      //根据id删除对应的权限
     async removeRightById(role,rightId){
      //弹窗提示用户是否要进行删除
     const confirmResult= await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        if(confirmResult!=='confirm'){
          return this.$message.info('取消了删除!') 
        }
       const {data:res}=await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
       if(res.meta.status!==200) return this.$message.error('删除权限失败')

       role.children=res.data
      },
      async showSetRightDialog(role){
        this.roleId=role.id
        //获取所有权限的数据
        const {data:res}=await this.$http.get('rights/tree')
        if(res.meta.status!==200) return  this.$message.error('获取权限信息错误')
        //把获取到的权限数据转存到数组中
        this.rightslist=res.data
        //递归获取三级节点的id
        this.getLeafKeys(role,this.defKeys);
        this.setRightDialogVisable=true
      },
      //通过递归的形式，获取角色的三级权限id并保存到数组中
      getLeafKeys(node,arr){
        if(!node.children){
          return arr.push(node.id)
        }
        node.children.forEach(item=>{
          this.getLeafKeys(item,arr)
        })
      },
      setRightDialogClosed(){
        this.defKeys=[]
      },
      //点击为角色分配权限
     async allotRights(){
        const keys=[...this.$refs.treeRef.getCheckedKeys(),...this.$refs.treeRef.getHalfCheckedKeys()]
        const idStr=keys.join(',')
        const {data:res}=  await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
        if(res.meta.status!==200){
          return this.$$message.error('更新权限失败')
        }
        this.$message.success('更新权限成功')
        this.getRoleList();
         this.setRightDialogVisable=false
    },
   async showEditDialog(id){
      //获取角色信息
      const {data:res}=await this.$http.get('roles/'+id)
      if(res.meta.status!==200) return this.$message.error('获取角色信息出错')
      this.editForm=res.data
      this.editDialogVisible=true
    },
    //监听修改用户描述对话框的关闭事件
    editDialogClosed(){
       this.$refs.editFormRef.resetFields()
    },
    
    //提交编辑的修改信息
    async editUserInfo(){
       const {data:res}=await this.$http.put('roles/'+this.editForm.roleId,{
         roleName:this.editForm.roleName,
         roleDesc:this.editForm.roleDesc
       })
      console.log(res);
       if(res.meta.status!==200) return this.$message.error('修改角色信息错误')
       //关闭对话框
       this.editDialogVisible=false
       //刷新数据列表
       this.getRoleList()
       //提示修改成功
       this.$message.success('修改成功')
    },
  async removeInfo(id){
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
      const {data:res}=await this.$http.delete('roles/'+id)
      console.log(res);
        if(res.meta.status!==200) return this.$message.error('删除角色失败')
        this.$message.success('成功山删除角色')
        this.getRoleList()
    },
    showAddCharDialog(){
      this.addDialogVisible=true
    },
  async addCharector(){
      const {data:res}=await this.$http.post('roles',{
        roleName :this.addForm.roleName,
        roleDesc :this.addForm.roleDesc
      })
      if(res.meta.status!==201) return this.$message.error('添加角色失败')
      this.$message.success('成功添加角色')
      this.addDialogVisible=false
      this.getRoleList()
    }  
    }
}
</script>

<style lang="less" scoped>
.el-tag{
  margin: 7px;
}
.bdtop{
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter{
  display: flex;
  align-items: center;
}
</style>