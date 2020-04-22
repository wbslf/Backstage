<template>
    <div>
      <!--面包屑导航区域--->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品分类</el-breadcrumb-item>
      </el-breadcrumb>

      <!--卡片视图区域-->
      <el-card>
        <el-row>
          <el-col>
            <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
          </el-col>
        </el-row>

        <!--表格-->
        <tree-table class="treeTable" :data="catelist" :columns="columns"
         :selection-type="false" :expand-type="false"
        show-index index-text="#" border :show-row-hover="false">
          <!--是否有效-->
          <template slot="isok" slot-scope="scope">
            <i class="el-icon-success" v-if="scope.row.cat_deleted === false"
              style="color:lightgreen;"></i>
            <i class="el-icon-error" v-else style="color:red;"></i>
          </template>
          <!--排序-->
          <template slot="order" slot-scope="scope">
            <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
            <el-tag type="success" size="mini" v-else-if="scope.row.cat_level===1">二级</el-tag>
            <el-tag type="warning" size="mini" v-else>三级</el-tag>
          </template>
          <!--操作-->
          <template slot="opt" slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.cat_id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCateById(scope.row.cat_id)">删除</el-button>
          </template>
        </tree-table>

        <!--分页区域-->
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[3,5,10,15]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          background>
        </el-pagination>
      </el-card>

      <!-- 添加分类的对话框 -->
      <el-dialog title="添加分类" :visible.sync="addDialogVisible" width="50%" @close="addCateDialogClosed">
        <!-- 添加分类的表单 -->
        <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
          <el-form-item label="分类名称：" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类：">
            <!-- options 用来指定数据源 -->
            <!-- props 用来指定配置对象 -->
            <el-cascader :options="parentCateList" :props="cascaderProps"
                 v-model="selectedKeys" @change="parentCateChanged"
                  :clearable="true" >
            </el-cascader>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
      </el-dialog>

      <!--编辑分类对话框-->
      <el-dialog title="修改分类" :visible.sync="editDialogVisible" width="50%"
                 @close="editDialogClosed">
        <!--内容主体区域--->
        <el-form :model="editForm" :rules="addCateFormRules" ref="editFormRef" label-width="70px">
          <el-form-item label="分类名称" >
            <el-input v-model="editForm.cat_name"></el-input>
          </el-form-item>
        </el-form>
        <!--底部按钮区域--->
        <span slot="footer">
          <el-button @click="editDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="editCate">确定</el-button>
        </span>
      </el-dialog>
    </div>
</template>

<script>
    export default {
      name: "Cate",
      data(){
        return{
          //获取商品分类列表的参数对象
          queryInfo: {
            type: 3,
            //当前的页数
            pagenum:1,
            //当前每页显示多少条数据
            pagesize:5
          },
          //商品分类的数据列表，默认为空
          catelist: [],
          //总数据条数
          total: 0,
          //为table指定列的定义
          columns:[{
            label:'分类名称',
            prop: 'cat_name'
          },{
            label:'是否有效',
            //将当前列定义为模板列
            type:'template',
            //当前一列使用的模板名称
            template:'isok'
          },{
            label:'排序',
            //将当前列定义为模板列
            type:'template',
            //当前一列使用的模板名称
            template:'order'
          },{
            label:'操作',
            //将当前列定义为模板列
            type:'template',
            //当前一列使用的模板名称
            template:'opt'
          }
          ],
          //控制添加分类对话框的显示与隐藏
          addDialogVisible:false,
          //添加分类的表单数据对象
          addCateForm:{
            //将要添加的分类的名称
            cat_name:'',
            //父级分类的id
            cat_pid: 0,
            //分类等级，默认添加一级分类
            cat_level:0
          },
          //添加分类表单的验证规则对象
          addCateFormRules:{
            cat_name:[
              { required:true, message:'请输入分类名称', trigger:'blur'}
            ]
          },
          //父级分类的列表
          parentCateList:[],
          //指定级联选择器的配置对象
          cascaderProps:{
            expandTrigger: 'hover',
            value:'cat_id',
            label:'cat_name',
            children:'children',
            checkStrictly:'true'
          },
          //选中的父级分类的id数组
          selectedKeys:[],
          //控制修改分类名对话框的显示与隐藏
          editDialogVisible:false,
          //查询到的分类对象
          editForm:{},
        }
      },
      created() {
        this.getCateList()
      },
      methods:{
        async getCateList(){
          const {data:res} = await this.$http.get('categories',{ params: this.queryInfo})
          if(res.meta.status !== 200) return this.$message.error('获取商品分类失败')
          this.catelist = res.data.result
          this.total = res.data.total
        },
        //监听pagesize改变的事件
        handleSizeChange(newSize){
          this.queryInfo.pagesize = newSize
          this.getCateList()
        },
        //监听页码值的改变事件
        handleCurrentChange(newPage){
          this.queryInfo.pagenum = newPage
          this.getCateList()
        },
        //点击按钮，展示添加分类的对话框
        showAddCateDialog(){
          //先获取父级分类的数据列表
          this.getParentCateList()
          //再展示对话框
          this.addDialogVisible = true
        },
        //获取父级分类的数据列表
        async getParentCateList() {
          const { data: res } = await this.$http.get('categories', {
            params: { type: 2 }
          })

          if (res.meta.status !== 200) {
            return this.$message.error('获取父级分类数据失败！')
          }

          //console.log(res.data)
          this.parentCateList = res.data
        },
        //选择项发生变化触发这个函数
        parentCateChanged() {
          console.log(this.selectedKeys)
          // 如果 selectedKeys 数组中的 length 大于0，证明选中的父级分类
          // 反之，就说明没有选中任何父级分类
          if (this.selectedKeys.length > 0) {
            // 父级分类的Id
            this.addCateForm.cat_pid = this.selectedKeys[
            this.selectedKeys.length - 1
              ]
            // 为当前分类的等级赋值
            this.addCateForm.cat_level = this.selectedKeys.length
            return
          } else {
            // 父级分类的Id
            this.addCateForm.cat_pid = 0
            // 为当前分类的等级赋值
            this.addCateForm.cat_level = 0
          }
        },
        // 点击按钮，添加新的分类
        addCate() {
          this.$refs.addCateFormRef.validate(async valid => {
            if (!valid) return
            const { data: res } = await this.$http.post(
              'categories',
              this.addCateForm
            )

            if (res.meta.status !== 201) {
              return this.$message.error('添加分类失败！')
            }

            this.$message.success('添加分类成功！')
            this.getCateList()
            this.addDialogVisible = false
          })
        },
        // 监听对话框的关闭事件，重置表单数据
        addCateDialogClosed() {
          this.$refs.addCateFormRef.resetFields()
          this.selectedKeys = []
          this.addCateForm.cat_level = 0
          this.addCateForm.cat_pid = 0
        },
        //监听修改分类名对话框的关闭事件
        editDialogClosed(){
          this.$refs.editFormRef.resetFields()
        },
        //修改分类名并提交
        editCate(){
          //预验证
          this.$refs.editFormRef.validate(async valid =>{
            if(!valid) return
            //可以发起修改用户的网络请求
            const {data:res} = await this.$http.put('categories/'+ this.editForm.cat_id,{cat_name:this.editForm.cat_name})
            if(res.meta.status !== 200) return this.$message.error('更新分类信息失败!')
            this.$message.success('更新分类信息成功!')
            this.editDialogVisible = false
            this.getCateList()
          })
        },
        //展示修改分类信息对话框
        async showEditDialog(id){
          const {data:res} = await this.$http.get('categories/' + id)
          if(res.meta.status !== 200) return this.$message.error('查询分类信息失败')
          this.editForm = res.data
          this.editDialogVisible = true
        },
        //删除分类对话框
        async removeCateById(id){
          const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).catch(err => err)
          //如果用户确认删除，返回字符串confirm
          //如果用户取消删除，返回字符串cancel
          if(confirmResult !== 'confirm'){
            return this.$message.info('已取消删除')
          }
          const {data:res} = await this.$http.delete('categories/' + id)
          if(res.meta.status !== 200) return this.$message.error('删除分类失败!')
          this.$message.success('删除分类成功!')
          this.getCateList()
        }
      }
    }
</script>

<style scoped>
.treeTable{
  margin-top: 15px;
}
.el-cascader{
  width: 100%;
}

</style>
