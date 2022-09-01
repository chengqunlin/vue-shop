<template>
    <div class="login_container">
        <div class="login_box">
            <div class="avatar">
                <img src="../assets/logo.png" alt="">
            </div>
            <el-form :model="loginForm" ref="LoginFormRef" :rules="loginFormRules" class="login_form">
                <el-form-item prop="username">
                    <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
                </el-form-item>
                <el-form-item prop="password">
                    <el-input v-model="loginForm.password" prefix-icon="iconfont icon-3702mima"></el-input>
                </el-form-item>
                <el-form-item class="btns">
                    <el-button type="primary" @click="login">登录</el-button>
                    <el-button type="info" @click="resetLoginForm">重置</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>

<script>
export default {
    props: {

    },
    data() {
        return {
            loginForm: {

            },
            loginFormRules: {
                username: [
                    { required: true, message: '请输入登录名', trigger: 'blur' },
                    {
                        min: 3,
                        max: 10,
                        message: '登录名长度在 3 到 10 个字符',
                        trigger: 'blur'
                    }
                ],
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    {
                        min: 6,
                        max: 15,
                        message: '密码长度在 6 到 15 个字符',
                        trigger: 'blur'
                    }
                ]
            }
        };
    },
    methods: {
        resetLoginForm() {
            this.$refs.LoginFormRef.resetFields()
        },
        login() {
            this.$refs.LoginFormRef.validate(async valid => {
                if (!valid) {
                    return
                }
                const { data: res } = await this.$http.post('login', this.loginForm)
                //   console.log(res);
                if (res.meta.status !== 200) {
                    return this.$message.error('登录失败:' + res.meta.msg) //console.log("登录失败:"+res.meta.msg)
                }

                this.$message.success('登录成功')
                console.log(res)
                //保存token
                window.sessionStorage.setItem('token', res.data.token)
                // 导航至/home
                this.$router.push('/home')
            })
        }

    },
    components: {

    },
};
</script>

<style scoped lang="less">
.login_container {
    height: 100%;
    background-color: #2b5b6b;

    .login_box {
        height: 300px;
        width: 450px;
        border-radius: 3px;
        background-color: #fff;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);

        .avatar {
            height: 130px;
            width: 130px;
            border: 1px solid #eee;
            border-radius: 50%;
            padding: 10px;
            box-shadow: 0 0 10px #ddd;
            position: absolute;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: #fff;

            img {
                width: 100%;
                height: 100%;
                border-radius: 50%;
                background-color: #eee;
            }
        }

        .login_form {
            position: absolute;
            bottom: 0;
            padding: 0 20px;
            width: 100%;
            box-sizing: border-box;

            .btns {
                display: flex;
                justify-content: flex-end;
            }
        }

    }
}
</style>
