<template>
    <div class="container" style="background-color: #fffaf0;min-height: 100vh;">
        <div class="top" style="line-height: 64px;padding-inline: 40px;background-color: #fef7e0;border-bottom: 1px solid #f5e6d3;">
            <el-tooltip class="item" effect="dark" content="返回首页" placement="bottom">
                <span class="last-page" @click="goBack">
                    <i class="el-icon-back"></i>
                </span>
            </el-tooltip>
            <span class="text" style="font-size: 18px;font-weight: 900;margin-left: 10px;color: #8b4513;">发布美食帖子</span>
        </div>
        <div class="buttom" style="margin-top: 6px;background-color: #fffaf0;">
            <div style="display: flex;background-color: white;border-radius: 8px;margin: 20px;border: 1px solid #f5e6d3;">
                <div style="width: 300px;padding: 20px;border-right: 1px solid #f5e6d3;background-color: #fef7e0;border-radius: 8px 0 0 8px;">
                    <div style="margin-bottom: 30px;">
                        <p style="font-size: 20px;font-weight: 600;color: #8b4513;margin-bottom: 10px;">菜系列表</p>
                        <el-select size="small" v-model="gourmet.categoryId" placeholder="选择菜系" style="width: 100%;"
                            :popper-class="'warm-yellow-dropdown'">
                            <el-option v-for="item in categories" :key="item.id" :label="item.name" :value="item.id"
                                style="font-size: 14px;">
                            </el-option>
                        </el-select>
                    </div>
                    <div style="margin-bottom: 30px;">
                        <p style="font-size: 20px;font-weight: 600;color: #8b4513;margin-bottom: 10px;">封面图片</p>
                        <el-upload class="avatar-uploader"
                            action="http://localhost:21090/api/food-share-sys/v1.0/file/upload" :show-file-list="false"
                            :on-success="handleCoverSuccess">
                            <img v-if="cover" :src="cover" style="width: 180px;height: 130px;border-radius: 4px;border: 1px solid #f5e6d3;">
                            <div v-else class="avatar-uploader-placeholder">
                                <i class="el-icon-plus avatar-uploader-icon"></i>
                                <span style="font-size: 12px;color: #a0522d;">上传封面</span>
                            </div>
                        </el-upload>
                    </div>
                    <div style="margin-bottom: 20px;">
                        <p style="font-size: 20px;font-weight: 600;color: #8b4513;margin-bottom: 10px;">公开权限</p>
                        <el-switch v-model="gourmet.isPublish" 
                                  active-color="#d2691e" 
                                  inactive-color="#c9a880"
                                  active-text="公开"
                                  inactive-text="私密">
                        </el-switch>
                    </div>
                </div>
                <div style="flex: 1;padding: 20px;background-color: #fffaf0;border-radius: 0 8px 8px 0;">
                    <div style="margin-bottom: 20px;padding-bottom: 20px;border-bottom: 1px solid #f5e6d3;">
                        <input v-model="gourmet.title" style="width: 100%;font-size: 28px;font-weight: 600;padding: 8px 15px;border: 1px solid #f5e6d3;border-radius: 6px;background-color: #fffaf0;color: #8b4513;transition: all 0.3s ease;" 
                            class="input-title" placeholder="请输入标题"
                            @focus="e => e.target.style.borderColor = '#f0c674'"
                            @blur="e => e.target.style.borderColor = '#f5e6d3'">
                    </div>
                    <div>
                        <Editor height="calc(100vh - 320px)" :receiveContent="gourmet.content" @on-receive="onReceive" />
                    </div>
                </div>
            </div>
            <div style="text-align: center;margin: 0 20px 20px;background-color: #fef7e0;border-radius: 8px;padding: 20px;border: 1px solid #f5e6d3;">
                <span class="edit-button" @click="postGourmet" style="padding: 10px 30px;font-size: 16px;">发布帖子</span>
            </div>
        </div>
    </div>
</template>
<script>
import Editor from "@/components/Editor"
export default {
    components: { Editor },
    name: "CreateGourmet",
    data() {
        return {
            gourmet: {},
            categories: [], // 美食帖子的数据集
            cover: null,
        }
    },
    created() {
        this.fetchFreshCategories();
    },
    methods: {
        // 发布美食
        postGourmet() {
            this.gourmet.cover = this.cover;
            this.$axios.post('/gourmet/save', this.gourmet).then(res => {
                if (res.data.code === 200) {
                    this.$notify({
                        duration: 1000,
                        title: '美食帖子新增',
                        message: '新增成功',
                        type: 'success'
                    });
                    this.goBack();
                }
            }).catch(error => {
                console.log("新增美食帖子异常：", error);
            });
        },
        // 封面上传回调函数
        handleCoverSuccess(res, file) {
            this.$notify({
                duration: 1500,
                title: '封面上传',
                message: res.code === 200 ? '上传成功' : '上传失败',
                type: res.code === 200 ? 'success' : 'error'
            });
            // 上传成功则更新用户头像
            if (res.code === 200) {
                this.cover = res.data;
            }
        },
        // 查询美食做法的信息
        fetchFreshCategories() {
            this.$axios.post('/category/query', {}).then(res => {
                if (res.data.code === 200) {
                    this.categories = res.data.data;
                    const allCategory = { id: null, name: '全部' };
                    // 头插
                    this.categories.unshift(allCategory);
                }
            }).catch(error => {
                console.log("查询美食帖子异常：", error);
            });
        },
        // 返回上一页
        goBack() {
            this.$router.go(-1);
        },
        onReceive(content) {
            this.gourmet.content = content;
        },
    }
};
</script>
<style scoped lang="scss">
.last-page:hover {
    background-color: #f5e6d3;
}

.last-page {
    background-color: #fef7e0;
    padding: 2px 4px;
    border-radius: 12px;
    border: 1px solid #f5e6d3;
    cursor: pointer;
    color: #8b4513;
    transition: all 0.3s ease;
}

.input-title:focus {
    outline: none;
    border-color: #f0c674 !important;
    box-shadow: 0 0 0 2px rgba(240, 198, 116, 0.2);
}

.input-title::placeholder {
    color: #c9a880;
    font-weight: normal;
}

/* 上传组件样式 */
:deep(.avatar-uploader) {
    .el-upload {
        border: 2px dashed #f5e6d3;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
        transition: border-color 0.3s;
        background-color: #fffaf0;
        
        &:hover {
            border-color: #f0c674;
        }
    }
    
    .avatar-uploader-placeholder {
        width: 180px;
        height: 130px;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }
    
    .avatar-uploader-icon {
        font-size: 28px;
        color: #c9a880;
        margin-bottom: 8px;
    }
}

/* 按钮样式 */
.edit-button {
    display: inline-block;
    padding: 10px 30px;
    background-color: #f0c674;
    color: #8b4513;
    border-radius: 20px;
    cursor: pointer;
    transition: all 0.3s ease;
    border: 1px solid #e8b85c;
    font-size: 16px;
    font-weight: 600;
    
    &:hover {
        background-color: #e8b85c;
        color: #fff;
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(240, 198, 116, 0.3);
    }
    
    &:active {
        transform: translateY(0);
    }
}

/* 选择框下拉菜单样式 */
.warm-yellow-dropdown {
    background-color: #fffaf0 !important;
    border: 1px solid #f5e6d3 !important;
    border-radius: 6px !important;
    
    .el-select-dropdown__item {
        color: #8b4513;
        
        &:hover, &.selected {
            background-color: #fef7e0;
        }
    }
}

/* 开关样式 */
:deep(.el-switch) {
    .el-switch__core {
        background-color: #c9a880;
        border-color: #c9a880;
        
        &::after {
            background-color: #fffaf0;
        }
    }
    
    &.is-checked {
        .el-switch__core {
            background-color: #d2691e;
            border-color: #d2691e;
        }
    }
    
    .el-switch__label {
        color: #8b4513;
        
        &.is-active {
            color: #d2691e;
        }
    }
}
</style>