<template>
    <div class="container">
        <!-- 原始密码 -->
        <div>
            <p class="dialog-title">*原始密码</p>
            <input class="dialog-input" type="password" v-model="oldPwd" placeholder="请输入" />
        </div>
        <!-- 新密码 -->
        <div>
            <p class="dialog-title">*新密码</p>
            <input class="dialog-input" type="password" v-model="newPwd" placeholder="请输入" />
        </div>
        <!-- 确认密码 -->
        <div>
            <p class="dialog-title">*确认密码</p>
            <input class="dialog-input" type="password" v-model="againPwd" placeholder="请输入" />
        </div>
        <div>
            <div style="margin-top: 20px;text-align: center;">
                <span class="edit-button" @click="postInfo">立即修改</span>
            </div>
        </div>
    </div>
</template>
<script>
import { clearToken } from "@/utils/storage"
export default {
    name: "ResetPwd",
    data() {
        return {
            oldPwd: '',
            newPwd: '',
            againPwd: ''
        }
    },
    methods: {
        // 提交密码信息
        async postInfo() {
            const userUpdatePwdDTO = {
                oldPwd: this.$md5(this.$md5(this.oldPwd)),
                newPwd: this.$md5(this.$md5(this.newPwd)),
                againPwd: this.$md5(this.$md5(this.againPwd))
            }
            const { data } = await this.$axios.put('/user/updatePwd', userUpdatePwdDTO);
            this.$notify({
                duration: 1000,
                title: '修改密码',
                message: data.code === 200 ? '修改成功，请重新登录' : data.msg,
                type: data.code === 200 ? 'success' : 'error'
            });
            if (data.code === 200) {
                clearToken();
                this.$router.push('/');
            }
        },
    }
};
</script>
<style scoped lang="scss">
.container {
    width: 500px;
    margin: 0 auto;
    padding: 20px;
    background-color: #fffaf0;
    border-radius: 8px;
    border: 1px solid #f5e6d3;
}

/* 标题样式 */
.dialog-title {
    color: #8b4513;
    font-size: 16px;
    font-weight: bold;
    margin-bottom: 8px;
}

/* 输入框样式 */
.dialog-input {
    width: 100%;
    padding: 12px 12px;
    line-height: 24px;
    border: 1px solid #f5e6d3;
    border-radius: 4px;
    background-color: #fffaf0;
    color: #8b4513;
    margin-bottom: 20px;
    transition: border-color 0.3s ease;
    font-size: 16px;
    
    &:focus {
        outline: none;
        border-color: #f0c674;
        box-shadow: 0 0 0 2px rgba(240, 198, 116, 0.2);
    }
    
    &::placeholder {
        color: #c9a880;
    }
}

/* 按钮样式 */
.edit-button {
    display: inline-block;
    padding: 8px 20px;
    background-color: #f0c674;
    color: #8b4513;
    border-radius: 20px;
    cursor: pointer;
    transition: all 0.3s ease;
    border: 1px solid #e8b85c;
    font-size: 14px;
    font-weight: 600;
    
    &:hover {
        background-color: #e8b85c;
        color: #fff;
        transform: translateY(-2px);
        box-shadow: 0 4px 8px rgba(240, 198, 116, 0.3);
    }
    
    &:active {
        transform: translateY(0);
    }
}
</style>