<template>
    <div>
      <!--面包屑导航区域--->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        <el-breadcrumb-item>添加商品</el-breadcrumb-item>
      </el-breadcrumb>

      <!--卡片视图区域-->
      <el-card>
        <!--提示区域-->
        <el-alert title="添加商品信息" type="info" center show-icon :closable="false"></el-alert>

        <!--步骤条区域-->
        <el-steps :space="200" :active="activeIndex-0" finish-status="success" align-center>
          <el-step title="基本信息"></el-step>
          <el-step title="商品参数"></el-step>
          <el-step title="商品属性"></el-step>
          <el-step title="商品图片"></el-step>
          <el-step title="商品内容"></el-step>
          <el-step title="完成"></el-step>
        </el-steps>

        <!--tap栏区域-->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
          <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="beforeTabLeave" @tab-click="tabClicked">
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
                <el-cascader :options="catelist" :props="cascaderProps"
                             v-model="addForm.goods_cat" @change="handleChanged"
                             :clearable="true" >
                </el-cascader>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品参数" name="1">
              <!--渲染表单的item项-->
              <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
                <!--复选框组-->
                <el-checkbox-group v-model="item.attr_vals">
                  <el-checkbox :label="cb" v-for="(cb,i) in item.attr_vals" :key="i" border></el-checkbox>
                </el-checkbox-group>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品属性" name="2">
              <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
                <el-input v-model="item.attr_vals"></el-input>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品图片" name="3">
              <!--action表示图片要上传到的api地址-->
              <el-upload
                action="http://127.0.0.1:8888/api/private/v1/upload"
                :on-preview="handlePreview"
                :on-remove="handleRemove"
                list-type="picture"
                :headers="headerObj"
                :on-success="handleSuccess">
                <el-button size="small" type="primary">点击上传</el-button>
              </el-upload>
            </el-tab-pane>
            <el-tab-pane label="商品内容" name="4">
              <!--富文本编辑器-->
              <quill-editor v-model="addForm.goods_introduce">
              </quill-editor>
              <!--添加商品按钮-->
              <el-button type="primary" class="addBtn" @click="add">添加商品</el-button>
            </el-tab-pane>
          </el-tabs>
        </el-form>
      </el-card>

      <!--图片预览-->
      <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
        <img :src="previewPath" class="previewImg">
      </el-dialog>
    </div>
</template>

<script>
  import _ from 'lodash'
  export default {
      name: "Add",
      data(){
        return{
          activeIndex:'0',
          addForm:{
            goods_name:'',
            //商品所属的分类数组
            goods_cat:[],
            goods_price:0,
            goods_number:0,
            goods_weight:0,
            goods_introduce:'',
            pics:[],
            attrs:[]
          },
          addFormRules:{
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
            ],
            goods_cat:[
              { required:true, message:'请选择商品分类', trigger:'blur'}
            ]
          },
          //商品分类列表
          catelist:[],
          //指定级联选择器的配置对象
          cascaderProps:{
            expandTrigger: 'hover',
            value:'cat_id',
            label:'cat_name',
            children:'children',
          },
          //动态参数数据
          manyTableData:[],
          //静态属性参数
          onlyTableData:[],
          //图片上传组件的headers请求头对象
          headerObj:{
            Authorization:
              window.sessionStorage.getItem('token')
          },
          previewPath:'',
          previewVisible:false

        }
      },
      created() {
        this.getCateList()
      },
      methods:{
        //获取所有的商品分类
        async getCateList(){
          const {data:res} = await this.$http.get('categories')
          if(res.meta.status !== 200) return this.$message.error('获取商品分类信息失败!')
          this.catelist = res.data
        },
        //级联选择框选中项变化，会触发这个函数
        handleChanged(){
          if(this.addForm.goods_cat.length !== 3){
            this.addForm.goods_cat = []
          }
        },
        beforeTabLeave(activeName,oldActiveName){
          if(oldActiveName === '0' && this.addForm.goods_cat.length !== 3){
            this.$message.error('请先选择商品分类')
            return false
          }
          return true
        },
        async tabClicked(){
          if(this.activeIndex === '1'){
            const {data:res} = await this.$http.get(`categories/${this.addForm.goods_cat[2]}/attributes`,
              {params:{sel:'many'}})
            if(res.meta.status !== 200) return this.$message.error('获取参数失败!')
            res.data.forEach(item =>{
              item.attr_vals = item.attr_vals ? item.attr_vals.split(' '):[]
            } )
            this.manyTableData = res.data
          }else if(this.activeIndex === '2'){
            const {data:res} = await this.$http.get(`categories/${this.addForm.goods_cat[2]}/attributes`,
              {params:{sel:'only'}})
            if(res.meta.status !== 200) return this.$message.error('获取属性失败!')
            this.onlyTableData = res.data
          }
        },
        //处理图片预览效果
        handlePreview(file){
          this.previewPath = file.response.data.url
          this.previewVisible = true
        },
        //处理移除图片操作
        handleRemove(file){
          //1.获取将要删除的图片的临时路径
          const filePath = file.response.data.tmp_path
          //2.从pics数组中找到这个图片对应的索引值
          const i = this.addForm.pics.findIndex(x => x.pic === filePath)
          //3.调用数组的splice方法，将图片信息对象从pics数组中移除
          this.addForm.pics.splice(i,1)
          //console.log(this.addForm)
        },
        //监听图片上传成功的事件
        handleSuccess(response){
          //1.拼接得到一个图片信息对象
          const picInfo = {pic:response.data.tmp_path}
          //2.将图片信息对象，push到pics数组中
          this.addForm.pics.push(picInfo)
        },
        //添加商品
        add(){
          this.$refs.addFormRef.validate(async valid => {
            if(!valid){
              return this.$message.error('请填写必要的表单项!')
            }
            //执行添加的业务逻辑
            const form = _.cloneDeep(this.addForm)
            form.goods_cat = form.goods_cat.join(',')
            //处理动态参数
            this.manyTableData.forEach(item =>{
              const newInfo = {attr_id:item.attr_id,attr_value:item.attr_vals.join(' ')}
              this.addForm.attrs.push(newInfo)
            })
            //处理静态属性
            this.onlyTableData.forEach(item =>{
              const newInfo = {attr_id:item.attr_id,attr_value:item.attr_vals}
              this.addForm.attrs.push(newInfo)
            })
            form.attrs = this.addForm.attrs
            //发起请求添加商品
            //商品的名称必须唯一
            const {data:res} = await this.$http.post('goods',form)
            if(res.meta.status !== 201) return this.$message.error('添加商品失败!')
            this.$message.success('添加商品成功!')
            this.$router.push('/goods')
          })
        }

      }
    }
</script>

<style scoped>
.el-checkbox{
  margin: 0 10px 0 0 !important;
}
.previewImg{
  width: 100%;
}
.addBtn{
  margin-top: 15px;
}
</style>
