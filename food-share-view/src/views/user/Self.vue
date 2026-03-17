<template>
    <div class="container">
        <!-- 头像 -->
        <div>
            <p class="dialog-title">*头像</p>
            <el-upload class="avatar-uploader" action="http://localhost:21090/api/food-share-sys/v1.0/file/upload" :show-file-list="false"
                :on-success="handleAvatarSuccess">
                <img v-if="userAvatar" :src="userAvatar" style="width: 88px;height: 88px;">
                <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
        </div>
        <!-- 昵称 -->
        <div>
            <p class="dialog-title">*昵称</p>
            <input class="dialog-input" v-model="userInfo.userName" placeholder="请输入" />
        </div>
        <!-- 邮箱 -->
        <div>
            <p class="dialog-title">*邮箱</p>
            <input class="dialog-input" v-model="userInfo.userEmail" placeholder="请输入" />
        </div>
        <!-- 账号状态 -->
        <div>
            <div>
                <p class="dialog-title">
                    *账号状态
                    <el-tooltip class="item" effect="dark" content="一经封号，不可登录，不可使用系统功能" placement="right">
                        <i class="el-icon-info"></i>
                    </el-tooltip>
                </p>
                <div>
                    <span v-if="!userInfo.isLogin" class="status-normal">
                        <i style="margin-right: 6px;" class="el-icon-circle-check"></i>正常</span>
                    <span v-else class="status-banned">
                        <i style="margin-right: 6px;" class="el-icon-warning-outline"></i>已被封禁</span>
                </div>
            </div>
            <div>
                <p class="dialog-title">
                    *留言状态
                    <el-tooltip class="item" effect="dark" content="禁言后，互动功能受限" placement="right">
                        <i class="el-icon-info"></i>
                    </el-tooltip>
                </p>
                <div>
                    <span v-if="!userInfo.isWord" class="status-normal">
                        <i style="margin-right: 6px;" class="el-icon-circle-check"></i>正常</span>
                    <span v-else class="status-banned">
                        <i style="margin-right: 6px;" class="el-icon-warning-outline"></i>已被封禁</span>
                </div>
            </div>
            <div style="margin-top: 20px;text-align: center;">
                <span class="edit-button" @click="postInfo">立即修改</span>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    name: "Self",
    data() {
        return {
            userInfo: {},
            userAvatar: '',
        }
    },
    created() {
        this.auth();
    },
    methods: {
        // 提交个人信息修改
        async postInfo() {
            const { userName, userEmail } = this.userInfo;
            const userUpdateDTO = {
                userAvatar: this.userAvatar,
                userName,
                userEmail
            }
            const { data } = await this.$axios.put('/user/update', userUpdateDTO);
            this.$notify({
                position: 'buttom-right',
                duration: 1000,
                title: '编辑个人信息',
                message: data.code === 200 ? '编辑成功' : '编辑失败',
                type: data.code === 200 ? 'success' : 'error'
            });
            // 返回首页
            if (data.code === 200) {
                this.auth();
            }
        },
        // 头像上传
        handleAvatarSuccess(res, file) {
            this.$notify({
                duration: 1500,
                title: '头像上传',
                message: res.code === 200 ? '上传成功' : '上传失败',
                type: res.code === 200 ? 'success' : 'error'
            });
            // 上传成功则更新用户头像
            if (res.code === 200) {
                this.userAvatar = res.data;
            }
        },
        // Token 检验 ,取得用户信息
        async auth() {
            const { data } = await this.$axios.get('/user/auth');
            if (data.code !== 200) { // Token校验异常
                this.$router.push('/');
            } else {
                this.userInfo = data.data;
                this.userAvatar = data.data.userAvatar;
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
    
    .el-icon-info {
        color: #a0522d;
        margin-left: 5px;
        cursor: pointer;
        
        &:hover {
            color: #d2691e;
        }
    }
}

/* 输入框样式 */
.dialog-input {
    width: 100%;
    padding: 8px 12px;
    border: 1px solid #f5e6d3;
    border-radius: 4px;
    background-color: #fffaf0;
    color: #8b4513;
    margin-bottom: 15px;
    transition: border-color 0.3s ease;
    
    &:focus {
        outline: none;
        border-color: #f0c674;
        box-shadow: 0 0 0 2px rgba(240, 198, 116, 0.2);
    }
    
    &::placeholder {
        color: #c9a880;
    }
}

/* 状态样式 */
.status-normal {
    font-size: 14px;
    color: #27a745;
    display: flex;
    align-items: center;
    
    .el-icon-circle-check {
        color: #27a745;
    }
}

.status-banned {
    font-size: 14px;
    color: #dc3545;
    display: flex;
    align-items: center;
    text-decoration: underline;
    text-decoration-style: dashed;
    
    .el-icon-warning-outline {
        color: #dc3545;
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

/* 上传组件样式 */
:deep(.avatar-uploader) {
    .el-upload {
        border: 2px dashed #f5e6d3;
        border-radius: 8px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
        transition: border-color 0.3s;
        background-color: #fffaf0;
        
        &:hover {
            border-color: #f0c674;
        }
    }
    
    .avatar-uploader-icon {
        font-size: 28px;
        color: #c9a880;
        width: 88px;
        height: 88px;
        line-height: 88px;
        text-align: center;
    }
    
    img {
        width: 88px;
        height: 88px;
        border-radius: 4px;
        border: 2px solid #f5e6d3;
    }
}

/* 提示框样式 */
:deep(.el-tooltip) {
    .el-tooltip__popper {
        background-color: #fef7e0;
        border: 1px solid #f5e6d3;
        color: #8b4513;
        
        &.is-dark {
            background-color: #8b4513;
            color: #fffaf0;
        }
        
        .popper__arrow {
            border-top-color: #8b4513;
            
            &::after {
                border-top-color: #8b4513;
            }
        }
    }
}
</style>