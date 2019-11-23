<template>
  <div>
    <!-- 面包屑 -->
      <el-breadcrumb separator="/">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
        <el-breadcrumb-item>用户列表</el-breadcrumb-item>
      </el-breadcrumb>

      <!-- 卡片视图 -->
      <el-card>
        <!-- 搜索与添加区域 -->
        <el-row :gutter="20">
          <!-- 用户名搜索 -->
          <el-col :span="8">
              <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
                  <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
              </el-input>
          </el-col>
          <!-- 用户添加 -->
          <el-col :span="4">
            <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
          </el-col>
        </el-row>

        <!-- 用户列表区域 -->
        <el-table :data="userlist" border stripe>
          <el-table-column type="index"></el-table-column>
          <el-table-column label="姓名" prop="username"></el-table-column>
          <el-table-column label="邮箱" prop="email"></el-table-column>
          <el-table-column label="电话" prop="mobile"></el-table-column>
          <el-table-column label="角色" prop="role_name"></el-table-column>
          <el-table-column label="状态" prop="mg_state">
            <template slot-scope="scope">
                <!-- {{scope.row}} -->
                <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"> </el-switch>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="180">
            <template slot-scope="scope">
                <!-- 修改按钮 -->
                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDalog(scope.row.id)"></el-button>
                <!-- 删除按钮 -->
                <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeUserById(scope.row.id)"></el-button>
                <!-- 分配角色按钮 -->
                <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                  <el-button type="warning" size="mini" icon="el-icon-setting"></el-button>
                </el-tooltip>
            </template>
          </el-table-column>
        </el-table>

          <!-- 分页 -->
          <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="queryInfo.pagenum"
              :page-sizes="[1,2,5,20]"
              :page-size="queryInfo.pagesize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="total">
          </el-pagination>
      </el-card>

      <!-- 添加用户对话框 -->
      <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        @close="addDialogClosed"
        width="50%"
        >
        <el-form :model="addForm" ref="addFormRef" :rules="addFormRules" label-width="80px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username" ></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password" ></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email" ></el-input>
          </el-form-item>
          <el-form-item label="手机号码"  prop="mobile">
            <el-input v-model="addForm.mobile" ></el-input>
          </el-form-item>
        </el-form>
        <!-- 对话框底部区 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户对话框 -->
        <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        @close="editDialogClosed"
        width="50%"
        >
        <el-form :model="editForm" ref="editFormRef" :rules="editFormRules" label-width="80px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email" ></el-input>
          </el-form-item>
          <el-form-item label="手机号码"  prop="mobile">
            <el-input v-model="editForm.mobile" ></el-input>
          </el-form-item>
        </el-form>
        <!-- 对话框底部区 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
      </el-dialog>
  </div>
</template>

<script>
import { userInfo } from 'os'
export default {
    data(){
      // 验证邮箱的规则
      var checkEmail =(rule,value,cb)=>{
        // 验证邮箱的正则表达式
        const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9])+/
        if(regEmail.test(value)){return cb()}
        cb(new Error('请输入正确的邮箱'))
      }

      // 验证手机号的规则
      var checkMobile =(rule,value,cb)=>{
        // 验证手机号的正则表达式
        const regMobile = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
        if(regMobile.test(value)){return cb()}
        cb(new Error('请输入正确的手机号'))
      }
      return {
          queryInfo:{
            // 搜索的内容
            query:'',
            // 当前页数
            pagenum:1,
            // 当前每页显示多少条数
            pagesize:2
          },
          userlist:[],
          total:0,
          // 添加用户对话框
          addDialogVisible:false,
          // 修改用户对话框
          editDialogVisible:false,
          // 添加的用户信息
          addForm:{
            username:'',
            password:'',
            email:'',
            mobile:'',
          },
          // 查询的用户信息
          editForm:{

          },
          // 添加表单的验证规则对象 
          addFormRules:{
            username:[
              {required:true,message:'请输入用户名',trigger:'blur'},
              {min:3,max:10,message:'用户名长度需要在3~10个字符之间',trigger:'blur'}
            ],
            password:[
              {required:true,message:'请输入密码',trigger:'blur'},
              {min:6,max:15,message:'请输入密码长度需要在6~15个字符之间',trigger:'blur'}
            ],
            email:[
              {required:true,message:'请输入邮箱',trigger:'blur'},
              {validator:checkEmail,trigger:'blur'}
            ],
            mobile:[
              {required:true,message:'请输入手机号码',trigger:'blur'},
              {validator:checkMobile,trigger:'blur'}
            ]
          },
          // 修改表单的验证规则对象
          editFormRules:{
            email:[
              {required:true,message:'请输入邮箱',trigger:'blur'},
              {validator:checkEmail,trigger:'blur'}
            ],
            mobile:[
              {required:true,message:'请输入手机号码',trigger:'blur'},
              {validator:checkMobile,trigger:'blur'}
            ]
          }
      }
    },
    created(){
      this.getUserList()
    },
    methods:{
      async getUserList(){
            const {data:res} =await this.$http.get('users',{params:this.queryInfo})
            // console.log(res);
            if(res.meta.status !==200)
            {
              return this.$message.error('获取用户列表失败！')
            }
            this.userlist = res.data.users
            this.total=res.data.total
            // console.log(this.userlist);
          },
      // 页数
      handleSizeChange(val){
        this.queryInfo.pagesize=val
        this.getUserList()
      },
      // 当前页
      handleCurrentChange(val){
        this.queryInfo.pagenum=val
        this.getUserList()
      },
      // 用户状态改变
      async userStateChanged(val){
        console.log(val);
      const {data: res} = await this.$http.put(`users/${val.id}/state/${val.mg_state}`)
      console.log(res);
      
        if(res.meta.status !== 200){
            val.mg_state =!val.mg_state
            return this.$message.error('更新用户状态失败')
        }
        this.$message.success('更新用户状态成功')
      },
      // 关闭用户添加对话框后,清空表单
      addDialogClosed(){
        this.$refs.addFormRef.resetFields()
      },
      // 点击确定按钮,添加用户
      addUser(){
        this.$refs.addFormRef.validate(async valid=>{
            if(!valid) return
            // 发起添加用户的网络请求
            const {data:res} = await this.$http.post('users', this.addForm)
            if(res.meta.status!==201)
            {
              this.$message.error('添加用户失败!')
            }
            this.$message.success('添加用户成功!')
            // 隐藏添加用户的对话框
            this.addDialogVisible=false
            // 重新获取用户列表数据
            this.getUserList()
        })
      },
      async showEditDalog(id){
        const {data:res}=await this.$http.get('users/'+id)
        if(res.meta.status !==200){
          return this.$message.error('查询用户信息失败!')
        }
        this.editForm=res.data
        // console.log(id);
        this.editDialogVisible=true
      },
      editUser(){
        this.$refs.editFormRef.validate(async valid=>{
            if(!valid) return
            // 发起修改用户的网络请求
            const {data:res} = await this.$http.put('users/' + this.editForm.id,
            {
              email:this.editForm.email,
              mobile :this.editForm.mobile
            })
            if(res.meta.status!==200)
            {
              this.$message.error('修改用户信息失败!')
            }
            this.$message.success('修改用户信息成功!')
            // 隐藏添加用户的对话框
            this.editDialogVisible=false
            // 重新获取用户列表数据
            this.getUserList()
        })
      },
      // 监听用户对话框的关闭事件
      editDialogClosed(){
        this.$refs.editFormRef.resetFields()
      },
      // 删除用户信息按钮
      async removeUserById(id){
      const confirmResult = await this.$confirm('将永久删除用户信息,是否继续?','提示',
        {
          confirmButtonText:'确定',
          cancelButtonText:'取消',
          type:'warning'
        }).catch(err=>err)
        // console.log(confirmResult); cancel取消  confirm确定
        if(confirmResult !=='confirm'){return this.$message.info('已取消删除')}

        const {data:res}=await this.$http.delete('users/' + id)
        if(res.meta.status !==200){
          return this.$message.error('删除用户失败!')
        }
        this.$message.success('删除用户成功!')
        this.getUserList()
      }
    }
}
</script>

<style lang="less" scoped>

</style>