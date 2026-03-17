<template>
    <div class="login-container">
        <div style="display: flex;justify-content: left;margin: 20px 0;">
            <Logo :bag="colorLogo" bag="rgb(51,51,51)" sysName="粤菜美食分享社区" />
        </div>
        <div class="login-panel">
            <div style="width: 300px;padding: 40px;display: flex;justify-content: center;align-items: center;border-radius: 20px 0 0 20px;overflow: hidden;background: linear-gradient(135deg, #FFF9F0 0%, #FFE9C8 100%);">
                <div style="width: 100%;height: 400px;display: flex;justify-content: center;align-items: center;border-radius: 15px;overflow: hidden;">
                    <img src="/bag.png" style="width: 100%;height: 100%;object-fit: contain;filter: drop-shadow(0 8px 20px rgba(0,0,0,0.1));" />
                </div>
            </div>
            <div class="right-login">
                <div style="text-align: center;margin-bottom: 30px;">
                    <h2 style="color: #5C3D2E;margin: 0 0 10px 0;font-size: 28px;font-weight: 700;">Hi，欢迎来到粤菜美食的世界</h2>
                    <p style="color: #8C6B4A;font-size: 16px;margin: 0;">品味地道粤菜，分享美食故事</p>
                </div>
                <div class="text">
                    <input v-model="act" class="act" placeholder="输入账号" />
                </div>
                <div class="text">
                    <input v-model="pwd" class="pwd" type="password" placeholder="输入密码" />
                </div>
                <div style="margin-top: 25px;">
                    <span class="login-btn" @click="login">立即登录</span>
                </div>
                <div class="tip">
                    <p>没有账号？<span class="no-act" @click="toDoRegister">点此注册</span></p>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
const DELAY_TIME = 1300;
import request from "@/utils/request.js";
import { setToken } from "@/utils/storage.js";
import md5 from 'js-md5';
import Logo from '@/components/Logo.vue';
export default {
    name: "Login",
    components: { Logo },
    data() {
        return {
            act: '',
            pwd: '',
            colorLogo: 'rgb(212, 165, 94)',
        }
    },
    methods: {
        // 跳转注册页面
        toDoRegister() {
            // 跳转
            this.$router.push('/register');
        },
        async login() {
            if (!this.act || !this.pwd) {
                this.$swal.fire({
                    title: '填写校验',
                    text: '账号或密码不能为空',
                    icon: 'error',
                    showConfirmButton: false,
                    timer: DELAY_TIME,
                });
                return;
            }
            const hashedPwd = md5(md5(this.pwd));
            const paramDTO = { userAccount: this.act, userPwd: hashedPwd };
            try {
                const { data } = await request.post(`user/login`, paramDTO);
                if (data.code !== 200) {
                    this.$swal.fire({
                        title: '登录失败',
                        text: data.msg,
                        icon: 'error',
                        showConfirmButton: false,
                        timer: DELAY_TIME,
                    });
                    return;
                }
                setToken(data.data.token);
                // 根据角色延迟跳转
                setTimeout(() => {
                    const { role } = data.data;
                    sessionStorage.setItem('role', role);
                    this.navigateToRole(role);
                }, DELAY_TIME);
            } catch (error) {
                console.error('登录请求错误:', error);
                this.$message.error('登录请求出错，请重试！');
            }
        },
        navigateToRole(role) {
            switch (role) {
                case 1:
                    this.$router.push('/admin');
                    break;
                case 2:
                    this.$router.push('/user');
                    break;
                default:
                    console.warn('未知的角色类型:', role);
                    break;
            }
        },
    }
};
</script>

<style lang="scss" scoped>
* {
    user-select: none;
}

.login-container {
    width: 100%;
    min-height: 100vh;
    background: linear-gradient(135deg, #FFF9F0 0%, #FFE9C8 100%);
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    position: relative;
    overflow: hidden;
    
    &::before {
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-image: 
            radial-gradient(circle at 20% 80%, rgba(255, 165, 0, 0.05) 0%, transparent 50%),
            radial-gradient(circle at 80% 20%, rgba(255, 140, 0, 0.05) 0%, transparent 50%);
        z-index: 0;
    }

    .login-panel {
        display: flex;
        border-radius: 25px;
        justify-content: space-between;
        height: auto;
        background: white;
        box-shadow: 0 20px 60px rgba(194, 145, 89, 0.3);
        position: relative;
        z-index: 1;
        overflow: hidden;
        border: 1px solid rgba(255, 233, 200, 0.5);

        .right-login {
            width: 350px;
            background-color: white;
            padding: 40px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            border-radius: 0 20px 20px 0;
        }

        .logo {
            margin: 10px 0 30px 0;
        }

        .act,
        .pwd {
            margin: 8px 0;
            height: 50px;
            line-height: 50px;
            width: 100%;
            font-size: 16px;
            font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
            font-weight: 500;
            padding: 0 8px;
            background-color: #FFF9F0;
            box-sizing: border-box;
            border: 2px solid #E8D4B9;
            border-radius: 10px;
            padding: 0 20px;
            margin-top: 20px;
            transition: all 0.3s ease;
            color: #5C3D2E;
        }

        .act:focus,
        .pwd:focus {
            outline: none;
            border: 2px solid #C28B59;
            background-color: #FFF5E6;
            box-shadow: 0 0 0 3px rgba(194, 145, 89, 0.1);
            transition: all 0.3s ease;
        }

        .act::placeholder,
        .pwd::placeholder {
            color: #B8A28C;
        }

        .role {
            display: inline-block;
            color: #C28B59;
            font-size: 14px;
            padding-right: 10px;
        }
    }
    
    .login-btn:hover {
        background: linear-gradient(135deg, #FF6347 0%, #FF4500 100%);
        transform: translateY(-2px);
        box-shadow: 0 8px 25px rgba(255, 99, 71, 0.4);
    }
    
    .login-btn {
        display: inline-block;
        text-align: center;
        border-radius: 12px;
        margin-top: 30px;
        height: 50px;
        line-height: 50px;
        width: 100%;
        background: linear-gradient(135deg, #FF7F50 0%, #FF6347 100%);
        font-size: 18px !important;
        font-weight: 600;
        border: none;
        color: white;
        padding: 0 !important;
        cursor: pointer;
        user-select: none;
        transition: all 0.3s ease;
        box-shadow: 0 4px 15px rgba(255, 127, 80, 0.3);
        letter-spacing: 1px;
    }

    .tip {
        margin: 30px 0 0 0;
        text-align: center;

        p {
            padding: 3px 0;
            margin: 0;
            font-size: 15px;
            color: #8C6B4A;

            i {
                margin-right: 3px;
            }

            span {
                color: #5C3D2E;
                border-radius: 2px;
                margin: 0 6px;
            }

            .no-act:hover {
                color: #FF7F50;
                cursor: pointer;
                text-decoration: underline;
                text-decoration-color: #FFD700;
            }

        }
    }

}
</style>