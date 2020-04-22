<template>
    <div>
      <!--面包屑导航区域--->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
      </el-breadcrumb>

      <!--卡片视图-->
      <el-card>
        <!--添加角色按钮区域-->
        <el-row>
          <el-col>
            <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
          </el-col>
        </el-row>

        <!--角色列表区域-->
        <el-table :data="rolelist" border stripe>
          <!--展开列-->
          <el-table-column type="expand">
            <template slot-scope="scope">
              <el-row :class="['bdbottom',i1===0 ? 'bdtop' : '','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                <!--渲染一级权限-->
                <el-col :span="5">
                  <el-tag closable @close="removeTagById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <!--渲染二、三级权限-->
                <el-col :span="19">
                  <!--通过for循环嵌套渲染二级权限-->
                  <el-row :class="[i2===0 ? '' : 'bdtop','vcenter']" v-for="(item2,i2) in item1.children"
                     :key="item2.id" >
                    <!--渲染二级权限-->
                    <el-col :span="6">
                      <el-tag type="success" closable @close="removeTagById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!--渲染三级权限-->
                    <el-col :span="18">
                      <el-tag type="warning" v-for="(item3,i3) in item2.children"
                       :key="item3.id" closable @close="removeTagById(scope.row,item3.id)">
                        {{item3.authName}}
                      </el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <!--索引列-->
          <el-table-column type="index" label="#"></el-table-column>
          <el-table-column label="角色名称" prop="roleName"></el-table-column>
          <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
          <el-table-column label="操作" width="300px">
            <template slot-scope="scope">
              <!--修改按钮-->
              <el-button type="primary" icon="el-icon-edit"
               size="mini" @click="showEditDialog(scope.row.id)">编辑
              </el-button>
              <!--删除按钮-->
              <el-button type="danger" icon="el-icon-delete"
                size="mini" @click="removeRoleById(scope.row.id)">删除
              </el-button>
              <!--分配权限按钮-->
              <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">
                分配权限
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-card>

      <!--添加用户对话框-->
      <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%"
                 @close="addDialogClosed">
        <!--内容主体区域--->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
          <el-form-item label="角色名" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!--底部按钮区域--->
        <span slot="footer">
          <el-button @click="addDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="addRole">确定</el-button>
        </span>
      </el-dialog>

      <!--修改角色对话框-->
      <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="50%"
                 @close="editDialogClosed">
        <!--内容主体区域--->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
          <el-form-item label="角色名" prop="roleName">
            <el-input v-model="editForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!--底部按钮区域--->
        <span slot="footer">
          <el-button @click="editDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="editRole">确定</el-button>
        </span>
      </el-dialog>

      <!--分配权限对话框-->
      <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%"
        @close="setRightDialogClosed">
        <!--内容主体区域--->
        <!--树形控件--->
        <el-tree :data="rightlist" :props="treeProps" show-checkbox
          node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
        <!--底部按钮区域--->
        <span slot="footer">
          <el-button @click="setRightDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="allotRights">确定</el-button>
        </span>
      </el-dialog>
    </div>
</template>

<script>
    export default {
      name: "Roles",
      data(){
        return{
          //所有角色列表数据
          rolelist: [],
          //控制添加角色对话框的显示与隐藏
          addDialogVisible: false,
          //添加用户表单数据
          addForm:{
            roleName:'',
            roleDesc:''
          },
          addFormRules: {
            roleName: [
              {required: true, message: '请输入角色名', trigger: 'blur'},
              {min: 3, max: 10, message: '用户名长度在3到10个字符之间', trigger: ['blur', 'change']}
            ],
            roleDesc: [
              {required: true, message: '请输入角色描述', trigger: 'blur'},
              {min: 3, max: 10, message: '用户名长度在3到10个字符之间', trigger: ['blur', 'change']}
            ]
          },
          //控制修改角色对话框的显示与隐藏
          editDialogVisible: false,
          //查询到的角色信息对象
          editForm:{},
          editFormRules:{
            roleName:[
              { required:true, message:'请输入角色名', trigger:'blur'},
              { min: 3, max: 10, message:'请输入正确的角色名', trigger:['blur','change']}
            ],
            roleDesc:[
              { required:true, message:'请输入角色描述', trigger:'blur'},
              { min: 3, max: 10,message:'请输入正确的角色描述', trigger:['blur','change']}
            ]
          },
          //控制分配权限对话框的显示与隐藏
          setRightDialogVisible: false,
          //查询到的权限列表数据
          rightlist:[],
          //树形控件的属性绑定对象
          treeProps:{
            label:'authName',
            children:'children'
          },
          //默认选择的节点id值数组
          defKeys:[],
          //当前即将分配权限的角色id
          roleId:''
        }
      },
      created() {
        this.getRoleList()
      },
      methods:{
        //获取所有角色列表
        async getRoleList(){
          const { data:res } = await this.$http.get('roles')
          if(res.meta.status !== 200) return this.$message.error('获取角色列表失败!')
          this.rolelist = res.data
        },
        //监听添加用户对话框的关闭事件
        addDialogClosed(){
          this.$refs.addFormRef.resetFields()
        },
        //添加新角色
        addRole(){
          //预验证
          this.$refs.addFormRef.validate(async valid =>{
            if(!valid) return
            //可以发起添加用户的网络请求
            const {data:res} = await this.$http.post('roles',this.addForm)
            if(res.meta.status !== 201) return this.$message.error('添加角色失败!')
            this.$message.success('添加角色成功!')
            this.addDialogVisible = false
            this.getRoleList()
          })
        },
        //监听修改用户对话框的关闭事件
        editDialogClosed(){
          this.$refs.editFormRef.resetFields()
        },
        //展示修改角色信息对话框
        async showEditDialog(id){
          const {data:res} = await this.$http.get('roles/' + id)
          if(res.meta.status !== 200) return this.$message.error('查询角色信息失败')
          this.editForm = res.data
          this.editDialogVisible = true
        },
        //修改用户信息并提交
        editRole(){
          //预验证
          this.$refs.editFormRef.validate(async valid =>{
            if(!valid) return
            //可以发起修改用户的网络请求
            const {data:res} = await this.$http.put('roles/'+ this.editForm.roleId,{roleName:this.editForm.roleName,roleDesc:this.editForm.roleDesc})
            if(res.meta.status !== 200) return this.$message.error('更新角色信息失败!')
            this.$message.success('更新角色信息成功!')
            this.editDialogVisible = false
            this.getRoleList()
          })
        },
        //删除角色对话框
        async removeRoleById(id){
          const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).catch(err => err)
          //如果用户确认删除，返回字符串confirm
          //如果用户取消删除，返回字符串cancel
          if(confirmResult !== 'confirm'){
            return this.$message.info('已取消删除')
          }
          const {data:res} = await this.$http.delete('roles/' + id)
          if(res.meta.status !== 200) return this.$message.error('删除角色失败!')
          this.$message.success('删除角色成功!')
          this.getRoleList()
        },
        //根据id删除权限对话框
        async removeTagById(role,rightId){
          const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).catch(err => err)
          //如果用户确认删除，返回字符串confirm
          //如果用户取消删除，返回字符串cancel
          if(confirmResult !== 'confirm'){
            return this.$message.info('已取消删除')
          }
          const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
          if(res.meta.status !== 200) return this.$message.error('删除权限失败!')
          this.$message.success('删除权限成功!')
          role.children = res.data
        },
        //展示分配权限对话框
        async showSetRightDialog(role){
          this.roleId = role.id
          //获取所有权限数据
          const {data:res} = await this.$http.get('rights/tree')
          if(res.meta.status !== 200) return this.$message.error('获取权限数据失败')
          this.rightlist = res.data
          //递归获取三级节点的id
          this.getLeafKeys(role,this.defKeys)
          this.setRightDialogVisible = true
        },
        //监听分配权限对话框关闭事件
        setRightDialogClosed(){
           this.defKeys = []
        },
        //通过递归的形式，获取角色下所有三级权限的id并保存到defKeys数组中
        getLeafKeys(node ,arr){
          //如果当前节点不包含children属性，则是三级节点
          if(!node.children){
            return arr.push(node.id)
          }
          node.children.forEach(item =>
           this.getLeafKeys(item,arr))
        },
        //点击为角色分配权限
        async allotRights(){
          const keys = [
            ...this.$refs.treeRef.getCheckedKeys(),
            ...this.$refs.treeRef.getHalfCheckedKeys()
          ]
          const idStr = keys.join(',')
          const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids: idStr})
          if(res.meta.status !== 200) return this.$message.error('分配权限失败!')
          this.$message.success('分配权限成功!')
          this.getRoleList()
          this.setRightDialogVisible = false
        }
      }
    }
</script>

<style scoped>
.el-tag{
  margin: 7px;
}
.bdtop{
  border-top:1px solid #eee;
}
.bdbottom{
  border-bottom:1px solid #eee;
}
.vcenter{
  display: flex;
  align-items: center;
}
</style>
