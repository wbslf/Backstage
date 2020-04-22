<template>
    <div>
      <!--面包屑导航区域--->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>订单管理</el-breadcrumb-item>
        <el-breadcrumb-item>订单列表</el-breadcrumb-item>
      </el-breadcrumb>

      <!--卡片视图区域-->
      <el-card>
        <!--搜索区域-->
        <el-row :gutter="20">
          <el-col :span="9">
            <!--搜索与添加区域-->
            <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getOrdersList">
              <el-button slot="append" icon="el-icon-search" @click="getOrdersList"></el-button>
            </el-input>
          </el-col>
        </el-row>

        <!--订单数据列表-->
        <el-table :data="orderslist" border stripe>
          <el-table-column label="#" type="index"></el-table-column>
          <el-table-column label="订单编号" prop="order_number"></el-table-column>
          <el-table-column label="订单价格(元)" prop="order_price" width="95px"></el-table-column>
          <el-table-column label="是否付款" prop="pay_status" width="95px">
            <template slot-scope="scope">
              <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
              <el-tag type="danger" v-else>未付款</el-tag>
            </template>
          </el-table-column>
          <el-table-column label="是否发货" prop="is_send" width="95px"></el-table-column>
          <el-table-column label="下单时间" prop="create_time" width="95px">
            <template slot-scope="scope">
              {{scope.row.create_time | dateFormat}}
            </template>
          </el-table-column>
          <el-table-column label="操作" width="120px">
            <template slot-scope="scope">
              <!--修改订单地址-->
              <el-button type="primary" icon="el-icon-edit"
                         size="mini" @click="showBox"></el-button>
              <!--查看物流信息-->
              <el-button type="success" icon="el-icon-location"
                         size="mini" @click="showProgressBox"></el-button>
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
      </el-card>

      <!--修改地址对话框-->
      <el-dialog title="修改地址" :visible.async="editDialogVisible" width="50%" @close="addressDialogClosed">
        <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
          <el-form-item label="省市区/县" prop="address1">
            <el-cascader :options="cityDate" v-model="addressForm.address1"
              :props="{expandTrigger: 'hover'}"></el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
            <el-input v-model="addressForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <!--底部按钮区域--->
        <span slot="footer">
          <el-button @click="editDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="editOrders">确定</el-button>
        </span>
      </el-dialog>

      <!--查看物流对话框-->
      <el-dialog title="查看物流" :visible.async="progressVisible" width="50%" @close="progressClosed">
        <el-timeline>
          <el-timeline-item v-for="(activity,index) in progressInfo" :key="index"
            :timestamp="activity.time">
            {{activity.context}}
          </el-timeline-item>
        </el-timeline>
      </el-dialog>
    </div>
</template>

<script>
  import cityData from './citydata'
  export default {
      name: "order",
      data(){
        return{
          //获取订单列表的参数对象
          queryInfo: {
            query: '',
            //当前的页数
            pagenum:1,
            //当前每页显示多少条数据
            pagesize:10
          },
          orderslist:[],
          total:0,
          editDialogVisible:false,
          addressForm:{
            address1:[],
            address2:''
          },
          addressFormRules:{
            address1:[
              { required:true, message:'请选择省市区/县', trigger:'blur'}
            ],
            address2:[
              { required:true, message:'请输入详细地址', trigger:'blur'}
            ]
          },
          cityDate:cityData,
          progressVisible:false,
          progressInfo:[]
        }
      },
      created() {
        this.getOrdersList()
      },
      methods:{
        async getOrdersList(){
          const {data:res} = await this.$http.get('orders',{params:this.queryInfo})
          if(res.meta.status !== 200) return this.$message.error('获取订单列表数据失败!')
          this.orderslist = res.data.goods
          this.total = res.data.total
        },
        //监听pagesize改变的事件
        handleSizeChange(newSize){
          this.queryInfo.pagesize = newSize
          this.getOrdersList()
        },
        //监听页码值的改变事件
        handleCurrentChange(newPage){
          this.queryInfo.pagenum = newPage
          this.getOrdersList()
        },
        //展示修改地址的对话框
        showBox(){
          this.editDialogVisible = true
        },
        //修改地址
        editOrders(){
          this.$message.success('修改地址成功!')
          this.editDialogVisible = false
        },
        //监听修改地址对话框的关闭事件
        addressDialogClosed(){
          this.$refs.addressFormRef.resetFields()
          this.editDialogVisible = false
        },
        //显示物流进度
        async showProgressBox(){
          const {data:res} = await this.$http.get('/kuaidi/1106975712662')
          if(res.meta.status !== 200) return this.$message.error('获取物流信息失败!')
          this.progressInfo = res.data
          this.progressVisible = true
        },
        progressClosed(){
          this.progressVisible = false
        }
      }
    }
</script>

<style scoped>
.el-cascader{
  width: 100%;
}
</style>
