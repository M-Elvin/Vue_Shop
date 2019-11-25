<template>
<div>
    <!-- 面包屑 -->
    <el-breadcrumb separator="/">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>

        <!-- 添加角色按钮区域 -->
        <el-row>
            <el-col>
                <el-button type="primary">添加角色</el-button>
            </el-col>
        </el-row>

        <!-- 角色列表区域 -->
        <el-table :data="rolelist" border stripe>
            <!-- 展开列 -->
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row :class="['bdbottom', i1 ===0? 'bdtop' :'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                        <!-- 渲染一级权限 -->
                        <el-col :span="5">
                            <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <!-- 渲染二级和三级权限 -->
                        <el-col :span="19">
                            <el-row :class="[i2 ===0 ? '':'bdtop','vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                <el-col :span="6">
                                    <el-tag closable @close="removeRightById(scope.row,item2.id)" type="success">{{item2.authName}}</el-tag>
                                    <i class="el-icon-caret-right"></i>
                                </el-col>
                                <el-col :span="18">
                                    <el-tag closable @close="removeRightById(scope.row,item3.id)" type="warning" v-for="(item3,i3) in item2.children" :key="item3.id">
                                        {{item3.authName}}
                                    </el-tag>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                    <pre>{{scope.row}}</pre>
                </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column width="300px" label="操作">
                <template slot-scope="scope">
                    <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                    <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                    <el-button size="mini" type="warning" icon="el-icon-setting" 
                    @click="showSetRightDialog(scope.row)"
                    >分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>

    <!-- 分配权限对话框 -->
     <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        @close="setRightDialogClosed"
        width="50%"
        >
        <el-tree :data="rightslist" :props="treeProps"
         show-checkbox node-key="id" default-expand-all
         :default-checked-keys="defKeys" ref="treeRef"></el-tree>
        <!-- 对话框底部区 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
      </el-dialog>
</div>
</template>

<script>
export default {
    data(){
        return{
            // 所有角色列表数据
            rolelist:[],
            //控制权限分配对话框的显隐
            setRightDialogVisible:false,
            // 所有权限的数据
            rightslist:[],
            // 树形控件的属性绑定对象
            treeProps:{
                label:'authName',
                children:'children'
            },
            defKeys:[],
            // 当前即将权限分配的ID
            roleId:''

        }
    },
    created(){
        this.getRolesList()
    },
    methods:{
        async getRolesList(){
            const {data:res} = await this.$http.get('roles')

            if(res.meta.status !== 200){
                return this.$message.error('获取角色列表失败!')
            }
            this.rolelist = res.data
            console.log(this.rolelist);
            
        },
        async removeRightById(role,rightId){
        const confirmResult = await this.$confirm('将永久删除该文件,是否继续?','提示',
        {
          confirmButtonText:'确定',
          cancelButtonText:'取消',
          type:'warning'
        }).catch(err=>err)
        // console.log(confirmResult); cancel取消  confirm确定
        if(confirmResult !=='confirm'){return this.$message.info('已取消删除')}

        const {data:res}=await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if(res.meta.status !==200){
          return this.$message.error('删除权限失败!')
        }
        this.$message.success('删除权限成功!')
        // this.getUserList()
        role.children=res.data
        },
        // 展示权限分配对话框
        async showSetRightDialog(role){
            this.roleId=role.id
            const {data:res}=await this.$http.get('rights/tree')

            if(res.meta.status !==200 )
            {
                return this.$message.error('获取权限数据失败')
            }
            // 把获取到的权限数据保存到data中
            this.rightslist = res.data
            this.getLeafKeys(role,this.defKeys)
            this.setRightDialogVisible=true
        },
        // 通过递归的形式,获取角色下所有三级权限的id,并保存到defKeys中
        getLeafKeys(node,arr){
            // 如果当前node节点不包含children属性,则是三级节点
            if(!node.children)
            {
                return arr.push(node.id)
            }
            node.children.forEach(item=>{
               this.getLeafKeys(item,arr)
            })
        },
        // 关闭后重置树状分配权限列表
        setRightDialogClosed(){
            this.defKeys=[]
        },
        async allotRights(){
            const keys=[
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]

            const idStr=keys.join(',')
            const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
            console.log(res);
            
            if(res.meta.status !==200 ){return this.$message.error('分配权限失败')}
            this.$message.success('分配权限成功')
            this.getRolesList()
            this.setRightDialogVisible = false
        }
    }
}
</script>

<style lang="less" scoped>
.el-tag{
    margin: 8px;
}
.bdtop{
    border-top: 1px solid #eee;
}
.bdbottom{
    border-bottom: 1px solid #eee;
}
.vcenter{
    display: flex;
    align-items: center
}
</style>