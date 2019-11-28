<template>
    <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator="/">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
        <el-row>
            <el-col>
                <el-button 
                @click="ShowAddDialog"
                type="primary">添加分类</el-button>
            </el-col>
        </el-row>

        <!-- 表格 -->
        <tree-table
        class="treeTable"
         :data="catelist" :columns="columns"
        :selection-type="false" :expand-type="false"
        show-index index-text="#" border :show-row-hover="false"
        >
            <!-- 是否有效 -->
            <template slot="isok" slot-scope="scope">
                <i class="el-icon-success"
                v-if="scope.row.cat_deleted===false" style=" color:lightgreen"></i>
                <i class="el-icon-error" v-else
                style="color:red"></i>
            </template>
            <!-- 排序 -->
            <template slot="order" slot-scope="scope">
                <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
                <el-tag type="success" size="mini"
                v-else-if="scope.row.cat_level===1">二级</el-tag>
                <el-tag type="warning" size="mini" v-else>三级</el-tag>
            </template>
            <!-- 操作 -->
            <template slot="opt" slot-scope="scope">
               <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
               <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            </template>
         </tree-table>

         <!-- 分页区域 -->
         <el-pagination @size-change="handleSizeChange"
         @current-change="handleCurrentChange"
         :current-page="queryInfo.pagenum"
         :page-sizes="[1,5,10,15]"
         :page-size="queryInfo.pagesize"
         layout="total,sizes,prev,pager,next,jumper" :total="total"></el-pagination>
    </el-card>

        <!-- 添加分类的对话框 -->
              <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        @close="addCateDialogClosed"
        width="50%"
        >
        <el-form :model="addCateForm" ref="addCateFormRef" :rules="addCateFormRules" label-width="80px">
          <el-form-item label="分类名称" prop="cat_name">
            <el-input v-model="addCateForm.cat_name" ></el-input>
          </el-form-item>
           <el-form-item label="父级分类" >
               <!-- options用来指定数据来源 -->
               <!-- props用来指定配置对象 -->
            <el-cascader
                expand-trigger="hover"
                :options="parentCateList"
                :props="cascaderProps"
                v-model="selectedKeys"
                @change="parentCateChanged"
                clearable change-on-select>
            </el-cascader>
          </el-form-item>
        </el-form>
        <!-- 对话框底部区 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
      </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
    return {
        queryInfo:{

            type:3,
            pagenum:1,
            pagesize:5
        },
        // 商品分类的数据列表,默认位空
        catelist:[],
        // 商品总条数
        total:0,
        // 为table指点列的定义
        columns:[
            {
                label:'分类名称',
                prop:'cat_name'
            },
            {
                label:'是否有效',
                type:'template',
                template:'isok'
            },
            {
                label:'排序',
                type:'template',
                template:'order'
            },
            {
                label:'操作',
                type:'template',
                template:'opt'
            }
        ],
        // 添加分类表单数据
        addCateForm:{
            // 分类的名称
            cat_name:'',
            // 父级分类的ID
            cat_pid:0,
            // 分类的等级,默认要添加的是1级分类
            cat_level:0
        },
        // 添加分类表单的验证规则对象
        addCateFormRules:{
            cat_name:[
                {required:true,message:'请输入分类名称',trigger:'blur'}
            ]
        },
        // 分类对话框
        addCateDialogVisible:false,
        // 父级分类的列表
        parentCateList:[],
     // 指定级联选择器的配置对象
      cascaderProps: {
        value:'cat_id',
        label:'cat_name',
        children:'children'
      },
          // 选中的父级分类的Id数组
      selectedKeys: []
    }
    },
    created() {
        this.getCateList()
    },
    methods: {
        async getCateList(){
            const {data:res} = await this.$http.get('categories',{ params :this.queryInfo })

            if(res.meta.status !==200 )
            {
                return this.$message.error('获取商品分类失败!')
            }
            // 获取商品分类
            this.catelist = res.data.result
            // 总条数赋值
            this.total=res.data.total
            // console.log(res);
            // console.log(this.catelist);
            // console.log(this.total);
        },
        // 监听 pagesize改变
        handleSizeChange(newSize){
            this.queryInfo.pagesize=newSize
            this.getCateList()
        },
        // 监听pagenum改变
        handleCurrentChange(newPage){
            this.queryInfo.pagenum=newPage
            this.getCateList()
        },
        // 展示添加分类对话框
        ShowAddDialog(){
            // 获取分类数据
            this.getParentCateList()
            this.addCateDialogVisible=true
        },
        // 获取分类的数据列表
        async getParentCateList(){
            const {data:res}=await this.$http.get('categories',{ params:{ type: 2 } })

            if(res.meta.status !==200){
                return this.$message.error('获取父级分类数据失败! ')
            }
            console.log(res.data);
            
            this.parentCateList=res.data
            console.log(this.parentCateList);
            
        },
        addDialogClosed(){

        },
         // 选择项发生变化触发这个函数
    parentCateChanged() {
      console.log(this.selectedKeys)
      // 如果 selectedKeys 数组中的 length 大于0，证明选中的父级分类
    //   反之，就说明没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的Id
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的Id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮，添加新的分类

    addCate() {
        console.log(this.addCateForm);
        
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)

        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }

        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
        // 监听对话框的关闭事件，重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }
    }
}
</script>

<style lang="less" scoped>
.treeTable{
    margin-top: 15px;
}
.el-cascader{
    width: 100%;
 
}
</style>
