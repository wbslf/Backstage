<template>
    <div>
      <!--面包屑导航区域--->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品列表</el-breadcrumb-item>
      </el-breadcrumb>

      <!--卡片视图区域-->
      <el-card>
        <!--搜索区域-->
        <el-row :gutter="20">
          <el-col :span="9">
            <!--搜索与添加区域-->
            <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
              <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
            </el-input>
          </el-col>
          <el-col :span="4">
            <el-button type="primary" @click="goAddpage">添加商品</el-button>
          </el-col>
        </el-row>

        <!--商品列表区域-->
        <el-table :data="goodslist" border stripe>
          <el-table-column label="#" type="index"></el-table-column>
          <el-table-column label="商品名称" prop="goods_name"></el-table-column>
          <el-table-column label="商品价格(元)" prop="goods_price" width="95px"></el-table-column>
          <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
          <el-table-column label="创建时间" prop="add_time" width="140px">
            <template slot-scope="scope">
              {{scope.row.add_time | dateFormat}}
            </template>
          </el-table-column>
          <el-table-column label="操作" width="120px">
            <template slot-scope="scope">
              <!--修改按钮-->
              <el-button type="primary" icon="el-icon-edit"
                         size="mini" @click="showEditDialog(scope.row.goods_id)"></el-button>
              <!--删除按钮-->
              <el-button type="danger" icon="el-icon-delete"
                         size="mini" @click="removeById(scope.row.goods_id)"></el-button>
            </template>
          </el-table-column>
        </el-table>

        <!--分页区域-->
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[10,15,20,30]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          background>
        </el-pagination>

        <!--修改商品对话框-->
        <el-dialog title="修改商品信息" :visible.sync="editDialogVisible" width="50%"
                   @close="editDialogClosed">
          <!--内容主体区域--->
          <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="editForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="editForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="editForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="editForm.goods_weight"></el-input>
            </el-form-item>
          </el-form>
          <!--底部按钮区域--->
          <span slot="footer">
          <el-button @click="editDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="editGoods">确定</el-button>
        </span>
        </el-dialog>

      </el-card>
    </div>
</template>

<script>
    export default {
      name: "List",
      data(){
        return{
          //获取商品列表的参数对象
          queryInfo: {
            query: '',
            //当前的页数
            pagenum:1,
            //当前每页显示多少条数据
            pagesize:10
          },
          //获取到的商品列表数据保存的数组
          goodslist:[],
          //总数据条数
          total:0,
          //控制添加商品对话框的显示与隐藏
          addGoodDialogVisible:false,
          editDialogVisible:false,
          editForm:{},
          editFormRules:{
            goods_name:[
              { required:true, message:'请输入商品名称', trigger:'blur'}
            ],
            goods_price:[
              { required:true, message:'请输入商品价格', trigger:'blur'}
            ],
            goods_weight:[
              { required:true, message:'请输入商品重量', trigger:'blur'}
            ],
            goods_number:[
              { required:true, message:'请输入商品数量', trigger:'blur'}
            ]
          }
        }
      },
      created() {
        this.getGoodsList()
      },
      methods:{
        //获取所有商品列表
        async getGoodsList(){
          const {data:res} = await this.$http.get('goods',{params:this.queryInfo})
          if(res.meta.status !== 200) return this.$message.error('获取商品列表数据失败!')
          this.goodslist = res.data.goods
          this.total = res.data.total
        },
        //监听pagesize改变的事件
        handleSizeChange(newSize){
          this.queryInfo.pagesize = newSize
          this.getGoodsList()
        },
        //监听页码值的改变事件
        handleCurrentChange(newPage){
          this.queryInfo.pagenum = newPage
          this.getGoodsList()
        },
        //显示修改商品信息对话框
        async showEditDialog(id){
          const {data:res} = await this.$http.get('goods/' + id)
          if(res.meta.status !== 200) return this.$message.error('查询商品信息失败')
          this.editForm = res.data
          console.log(this.editForm)
          this.editDialogVisible = true
        },
        //删除商品
        async removeById(id){
          const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).catch(err => err)
          //如果用户确认删除，返回字符串confirm
          //如果用户取消删除，返回字符串cancel
          if(confirmResult !== 'confirm'){
            return this.$message.info('已取消删除')
          }
          const {data:res} = await this.$http.delete('goods/' + id)
          if(res.meta.status !== 200) return this.$message.error('删除商品失败!')
          this.$message.success('删除商品成功!')
          this.getGoodsList()
        },
        //新增商品处理函数
        goAddpage(){
          this.$router.push('/goods/add')
        },
        //监听修改用户对话框的关闭事件
        editDialogClosed(){
          this.$refs.editFormRef.resetFields()
        },
        //编辑商品信息并提交
        editGoods(){
          //预验证
          this.$refs.editFormRef.validate(async valid =>{
            if(!valid) return
            //可以发起修改用户的网络请求
            const {data:res} = await this.$http.put('goods/'+ this.editForm.goods_id,
              {goods_name:this.editForm.goods_name,goods_price:this.editForm.goods_price,
                goods_number:this.editForm.goods_number,goods_weight:this.editForm.goods_weight,
                goods_introduce:'',pics:{}, attrs:[],goods_cat:this.editForm.goods_cat})
            console.log(res.meta.status)
            if(res.meta.status !== 200) return this.$message.error('更新商品信息失败!')
            this.$message.success('更新商品信息成功!')
            this.editDialogVisible = false
            this.getGoodsList()
          })
        }
      }
    }
</script>

<style scoped>

</style>
