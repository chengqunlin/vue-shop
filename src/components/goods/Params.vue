<template>
    <div>
        <h3>分类参数</h3>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>分类参数</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 警告区域 :closable="false"(是否展示“X”号) show-icon(显示图标) -->
            <el-alert title="注意：只允许为第三级分类设置相关参数" type="warning" :closable="false" show-icon>
            </el-alert>

            <!-- 选择商品分类区域 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类：</span>
                    <!-- 选择商品分类的级联选择框 -->
                            <el-cascader expandTrigger='hover'
                             v-model="selectedCateKeys" :options="cateList" :props="cateProps" 
                             @change="handleChange" clearable>
                             </el-cascader>
                </el-col>
                <el-col></el-col>
            </el-row>

            <!-- tab页签区域 -->
            <el-tabs v-model="activeName" @tab-click="handleTabClick">
            <!-- 添加动态参数的面板 将标签页改为many -->
                <el-tab-pane label="动态参数" name="many">
                    <el-button size="mini" type="primary" :disabled="isButtonDisabled" @click="showAddDialog">添加参数</el-button>
                    <!-- 动态参数表格 -->
                    <el-table :data="manyTableData" border stripe>
                        <!-- 展开行 -->
                        <el-table-column type="expand"></el-table-column>
                        <!-- 索引列 -->
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <!-- 添加静态属性的面板 将标签页改为only -->
                <el-tab-pane label="静态属性" name="only">
                    <el-button size="mini" type="primary" :disabled="isButtonDisabled">添加属性</el-button>
                    <!-- 静态属性表格 -->
                    <el-table :data="onlyTableData" border stripe>
                    <!-- 展开行 -->
                        <el-table-column type="expand"></el-table-column>
                        <!-- 索引列 -->
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>

        <el-dialog :title="'添加'+titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
            <!-- 添加表单 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addParams">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 修改参数或属性对话框 -->
        <el-dialog :title="'修改'+titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <!-- 添加表单 -->
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="editForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editParams">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    props: {

    },
    data() {
        return {
            cateList:[],
            //用户在级联下拉菜单中选中的分类id
            selectedCateKeys:[],
            //配置级联菜单中数据如何展示
            cateProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            //tab页签激活显示的页签项
            activeName: 'many',
            //用来保存动态参数数据
            manyTableData: [],
            //用来保存静态属性数据
            onlyTableData: [],

            addDialogVisible: false,
            //添加参数的表单数据对象
            addForm: {
                attr_name: ''
            },
            //添加表单验证规则
            addFormRules: {
                attr_name: [{ required: true, message: '请输入名称', trigger: 'blur' }]
            },
            editDialogVisible:false,
            //修改参数.属性对话框中的表单
            editForm:{
                attr_name:''
            },
            //修改表单的验证规则
            editFormRules:{
                attr_name:[
                { required: true, message: '请输入名称', trigger: 'blur' }
                ]
            }
        };
    },
     created() {
      this.getCateList()
    },
    methods: {
        async getCateList(){
        //获取所有的商品分类列表
        const { data: res } = await this.$http.get('categories')

        if (res.meta.status !== 200) {
            return this.$message.error('获取分类数据失败')
        }
        //将数据列表赋值给cateList
        this.cateList = res.data
        // //保存总数据条数
        // this.total = res.data.total
        console.log(res.data);
      },
    async handleChange() {
        //当用户在级联菜单中选择内容改变时触发
        console.log(this.selectedCateKeys)
        //发送请求，根据用户选择的三级分类和面板获取参数数据
        const { data: res } = await this.$http.get(
            `categories/${this.cateId}/attributes`,
            { params: { sel: this.activeName } }
        )
        if (res.meta.status !== 200) {
            return this.$message.error('获取参数列表数据失败')
        }

        console.log(res.data)
        if (this.activeName === 'many') {
            //获取的是动态参数
            this.manyTableData = res.data
        } else if (this.activeName === 'only') {
            //获取的是静态属性
            this.onlyTableData = res.data
        }
    },
    handleTabClick() {
        console.log(this.activeName)
        this.handleChange()
    },
    addParams() {
      //当用户点击对话框中的确定时，校验表单
      this.$refs.addFormRef.validate(async valid => {
        //校验不通过，return
        if (!valid) return
        //校验通过，发送请求完成添加参数或者属性
        const { data: res } = this.$http.post(`categories/${this.cateId}/attributes`,
          { 
            attr_name: this.addForm.attr_name, 
            attr_sel: this.activeName,
            attr_vals: "a,b,c" 
          }
        )

        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加' + this.titleText + '数据失败')
        }
        this.$message.success('添加' + this.titleText + '数据成功')
        this.addDialogVisible = false
        this.getCateList()
      })
    },
    addDialogClosed(){
        this.$refs.addFormRef.resetFields()
    },

    showAddDialog(){
        this.addDialogVisible = true
    },

    async showEditDialog(attr_id){
      //发起请求获取需要修改的那个参数数据
      const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,
      {params:{ attr_sel:this.activeName }})
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数数据失败')
      }
      this.editForm = res.data;
      //显示修改参数.属性对话框
      this.editDialogVisible = true;
    },
    editDialogClosed(){
      //当关闭修改参数.属性对话框时
      this.$refs.editFormRef.resetFields()
    },
    editParams(){
      //验证表单
      this.$refs.editFormRef.validate(async valid => {
        if(!valid) return;

        //发送请求完成修改
        const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
        {attr_name:this.editForm.attr_name,attr_sel:this.activeName})

        if (res.meta.status !== 200) {
          return this.$message.error('获取参数数据失败')
        }
        this.$message.success('修改' + this.titleText + '数据成功')
        this.editDialogVisible = false
        this.handleChange();
      })
    },
    async removeParams(attr_id){
    //根据id删除对应的参数或属性
    //弹窗提示用户是否要删除
    const confirmResult = await this.$confirm(
        '请问是否要删除该'+this.titleText,
        '删除提示',
        {
        confirmButtonText: '确认删除',
        cancelButtonText: '取消',
        type: 'warning'
        }
    ).catch(err => err)
    //如果用户点击确认，则confirmResult 为'confirm'
    //如果用户点击取消, 则confirmResult获取的就是catch的错误消息'cancel'
    if (confirmResult != 'confirm') {
        return this.$message.info('已经取消删除')
    }

    //没有取消就是要删除，发送请求完成删除
    const {data:res} = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)

    if (res.meta.status !== 200) {
        return this.$message.error('删除参数数据失败')
    }
    this.$message.success('删除' + this.titleText + '数据成功')
    this.handleChange()
    }

    },
    computed: {
        //添加计算属性用来获取按钮禁用与否
        isButtonDisabled() {
            return this.selectedCateKeys.length !== 3
        },
        //获取选中的三级分类id
        cateId() {
        if (this.selectedCateKeys.length === 3) {
            return this.selectedCateKeys[this.selectedCateKeys.length - 1]
        }
        return null
        },
        titleText(){
            if(this.activeName==='many'){
                return '动态参数'
            }else{
                return '静态参数'
            }
        }
    },
    components: {

    },
};
</script>

<style scoped lang="less">

</style>
