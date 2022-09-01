<template>
    <div>
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item><a href="/">用户管理</a></el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card class="box-card">
            <el-row :gutter="20">
                <el-col :span="10">
                    <el-input  v-model="queryInfo.query" placeholder="请输入内容" clearable @clear="getUserList">
                        <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                    </el-input></el-col>
                <el-col :span="4">
                     <el-button type="primary" @click="showAddDialog">添加用户</el-button>
                </el-col>
            </el-row>
            <el-table :data="userList" border stripe>
                <el-table-column type="index" label="序号" width="80"></el-table-column>
                <el-table-column label="姓名" prop="username"></el-table-column>
                <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="状态">
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)"></el-switch>
                    </template>
                </el-table-column>
                 <el-table-column label="操作" width="180px">
                    <template slot-scope="scope">
                        <!-- 修改 -->
                        <el-button type="primary" icon="el-icon-edit" size='mini' @click="showEditDialog(scope.row.id)"></el-button>
                        <!-- 删除 -->
                        <el-button type="danger" icon="el-icon-delete" size='mini' @click="removeUserById(scope.row.id)"></el-button>
                        <!-- 分配角色 -->
                        <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size='mini' @click="setRole(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>

            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[1, 2, 5, 10]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>

        <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        width="50%" @close="addDialogClose">
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
        </el-dialog>

        <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        width="50%" @close="editDialogClosed">
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
                <el-form-item label="用户名">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
        </el-dialog>

        <!-- 分配角色对话框 -->
        <el-dialog title="分配角色" :visible.sync="setRoleDialogVisible" width="50%" @close="setRoleDialogClosed">
            <div>
            <p>当前的用户:{{userInfo.username}}</p>
            <p>当前的角色:{{userInfo.role_name}}</p>
            <p>分配新角色:
                <el-select v-model="selectedRoleId" placeholder="请选择角色">
                <!-- :label 显示文本，:value 选中值 -->
                <el-option v-for="item in rolesList" :key="item.id" :label="item.roleName" :value="item.id">
                </el-option>
                </el-select>
            </p>
            </div>
            <span slot="footer" class="dialog-footer">
            <el-button @click="setRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    props: {

    },
    data() {
         var checkEmail = (rule, value, cb) => {
            const regEmail = /^\w+@\w+(\.\w+)+$/
            if (regEmail.test(value)) {
            return cb()
            }
            //返回一个错误提示
            cb(new Error('请输入合法的邮箱'))
        }
        //验证手机号码的规则
        var checkMobile = (rule, value, cb) => {
            const regMobile = /^1[34578]\d{9}$/
            if (regMobile.test(value)) {
            return cb()
            }
            //返回一个错误提示
            cb(new Error('请输入合法的手机号码'))
        }
        return {
            queryInfo:{
                query:'',
                pagenum: 1,
                pagesize: 5
            },
            userList:[],
            total:0,
            addDialogVisible:false,
            addForm: {
                username: '',
                password: '',
                email: '',
                mobile: ''
            },
            addFormRules: {
                username: [
                    { required: true, message: '请输入用户名称', trigger: 'blur' },
                    {
                    min: 3,
                    max: 10,
                    message: '用户名在3~10个字符之间',
                    trigger: 'blur'
                    }
                ],
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    {
                    min: 6,
                    max: 15,
                    message: '用户名在6~15个字符之间',
                    trigger: 'blur'
                    }
                ],
                email: [
                    { required: true, message: '请输入邮箱', trigger: 'blur' },
                    { validator:checkEmail, message: '邮箱格式不正确，请重新输入', trigger: 'blur'}
                ],
                mobile: [
                    { required: true, message: '请输入手机号码', trigger: 'blur' },
                    { validator:checkMobile, message: '手机号码不正确，请重新输入', trigger: 'blur'}
                ]
            },
            editDialogVisible: false,
            //修改用户的表单数据
            editForm: {
                username: '',
                email: '',
                mobile: ''
            },
            //修改表单的验证规则对象
            editFormRules: {
                email: [
                    { required: true, message: '请输入邮箱', trigger: 'blur' },
                    {
                    validator: checkEmail,
                    message: '邮箱格式不正确，请重新输入',
                    trigger: 'blur'
                    }
                ],
                mobile: [
                    { required: true, message: '请输入手机号码', trigger: 'blur' },
                    {
                    validator: checkMobile,
                    message: '手机号码不正确，请重新输入',
                    trigger: 'blur'
                    }
                ]
            },

            setRoleDialogVisible:false,
            userInfo:{},
            //保存所有的角色信息
            rolesList:[],
            //保存用户选中的角色id
            selectedRoleId:''
        }
    },
    created() {
        this.getUserList()
    },
    methods: {
        async getUserList(){
            const { data: res } = await this.$http.get('users', {
                params: this.queryInfo
            })
            if (res.meta.status !== 200){
                console.log(res.meta.status)
                return this.$message.error('获取用户列表失败')
            }

            this.userList = res.data.users;
            this.total = res.data.total;
            console.log(res)
        },
        handleSizeChange(pagesize){
            this.queryInfo.pagesize = pagesize
            this.getUserList()
        },
        handleCurrentChange(pagenum){
            this.queryInfo.pagenum = pagenum
            this.getUserList()
        },
        async userStateChange(row){
            const { data: res } = await this.$http.put(
                `users/${row.id}/state/${row.mg_state}`
            )
            //如果返回状态为异常状态则报错并返回
            if (res.meta.status !== 200) {
                row.mg_state = !row.mg_state
                return this.$message.error('修改状态失败')
            }
            this.$message.success('更新状态成功')
        },
        showAddDialog(){
            this.addDialogVisible = true
        },
        addDialogClose(){
            this.$refs.addFormRef.resetFields()
        },
        addUser(){
            this.$refs.addFormRef.validate( async valid => {
                if(!valid) return this.$message.error("请填写完整用户信息");
                //发送请求完成添加用户的操作
                const {data:res} = await this.$http.post("users",this.addForm)
                //判断如果添加失败，就做提示
                if (res.meta.status !== 201)
                    return this.$message.error('添加用户失败')
                //添加成功的提示
                this.$message.success("添加用户成功")
                //关闭对话框
                this.addDialogVisible = false
                //重新请求最新的数据
                this.getUserList()
            })
        },
        async showEditDialog(id) {
            //发送请求根据id获取用户信息
            const { data: res } = await this.$http.get('users/' + id)
            //判断如果添加失败，就做提示
            if (res.meta.status !== 200) return this.$message.error('获取用户信息失败')
            //将获取到的数据保存到数据editForm中
            this.editForm = res.data
            //显示弹出窗
            this.editDialogVisible = true
        },
        editDialogClosed(){
            //对话框关闭之后，重置表达
            this.$refs.editFormRef.resetFields()
        },
        editUser() {
            //用户点击修改表单中的确定按钮之后，验证表单数据
            this.$refs.editFormRef.validate(async valid => {
                if (!valid) return this.$message.error('请填写完整用户信息')
                //发送请求完成修改用户的操作
                const { data: res } = await this.$http.put(
                    'users/' + this.editForm.id,
                    this.editForm
                )
                //判断如果修改失败，就做提示
                if (res.meta.status !== 200) return this.$message.error('修改用户失败')
                //修改成功的提示
                this.$message.success('修改用户成功')
                //关闭对话框
                this.editDialogVisible = false
                //重新请求最新的数据
                this.getUserList()
            })
        },
        removeUserById(id){
            this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(async () => {
                const {data:res} = await this.$http.delete('users/'+id)
                //判断如果删除失败，就做提示
                if (res.meta.status !== 200) return this.$message.error('删除用户失败')
                //修改成功的提示
                this.$message.success('删除用户成功')
                //重新请求最新的数据
                this.getUserList()
            }).catch(() => {
            this.$message({
                type: 'info',
                message: '已取消删除'
            });          
            });
        },
        async setRole( userInfo ){
            //保存起来以供后续使用
            this.userInfo = userInfo;
            //获取所有的角色信息，以备下拉列表使用
            //发送请求根据id完成删除操作
            const { data: res } = await this.$http.get('roles')
            //判断如果删除失败，就做提示
            if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
            
            this.rolesList = res.data;
            //展示分配角色对话框
            this.setRoleDialogVisible = true;
        },
        async saveRoleInfo(){
        //当用户点击确定按钮之后
        //判断用户是否选择了需要分配的角色
        if(!this.selectedRoleId){
            return this.$message.error('请选择需要分配的角色')
        }
        //发送请求完成分配角色的操作
        const {data:res} = await this.$http.put(`users/${this.userInfo.id}/role`,{rid:this.selectedRoleId})

        //判断如果删除失败，就做提示
        if (res.meta.status !== 200)
            return this.$message.error('分配角色失败')

        this.$message.success('分配角色成功')
        this.getUserList();
        //关闭对话框
        this.setRoleDialogVisible = false
        },
        setRoleDialogClosed(){
        //当关闭对话框的时候，重置下拉框中的内容
        this.selectedRoleId = ''
        this.userInfo = {}
        }
    },
    components: {

    },
};
</script>

<style scoped lang="less">

</style>
