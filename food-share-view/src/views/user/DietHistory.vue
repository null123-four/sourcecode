<template>
    <div style="background-color: #FFF9F0;padding: 20px;min-height: 100vh;">
        <!-- 添加菜单卡片 -->
        <div class="record" @click="recordGourmet">
            <div style="text-align: center;">
                <div style="width: 60px;height: 60px;border-radius: 50%;background: linear-gradient(135deg, #FFE9C8 0%, #FFD8A8 100%);display: flex;align-items: center;justify-content: center;margin: 0 auto 20px;">
                    <i class="el-icon-food" style="font-size: 30px;color: #C28B59;"></i>
                </div>
                <span style="font-size: 24px;font-weight: bold;color: #5C3D2E;">
                    添加菜单
                </span>
                <p style="color: #8C6B4A;margin-top: 10px;font-size: 16px;">点击这里记录您的美食用餐</p>
            </div>
        </div>
        
        <h3 style="font-size: 24px;color: #5C3D2E;font-weight: bold;margin: 30px 0 20px;padding-bottom: 10px;border-bottom: 2px solid #E8D4B9;">我的菜单</h3>
        
        <div>
            <div v-if="dietHitoryList.length === 0">
                <el-empty description="快补充菜单吧" :image-size="120"></el-empty>
            </div>
            <div v-else>
                <div class="item-diet" v-for="(dietHistory, index) in dietHitoryList" :key="index">
                    <div style="display: flex;align-items: center;justify-content: space-between;margin-bottom: 8px;">
                        <div style="font-size: 22px;font-weight: bold;color: #5C3D2E;display: flex;align-items: center;gap: 8px;">
                            <i class="el-icon-food" style="color: #C28B59;"></i>
                            {{ getCookbookName(dietHistory) }}
                        </div>
                        <div style="cursor: pointer;color: #8C6B4A;transition: all 0.3s;" @click="removeCookbook(dietHistory)" class="remove-icon">
                            <i class="el-icon-circle-close" style="font-size: 20px;"></i>
                        </div>
                    </div>
                    <div style="color: #8C6B4A;font-size: 15px;line-height: 1.5;padding: 12px;background-color: #FFF9F0;border-radius: 8px;border: 1px solid #E8D4B9;">
                        <span style="display: inline-block;background-color: #FFF5E6;padding: 6px 12px;border-radius: 6px;color: #A8723A;font-weight: 500;">
                            {{ dietHistory.value }}g
                        </span>
                        <span style="margin-left: 12px;">{{ dietHistory.detail || '暂无备注' }}</span>
                    </div>
                </div>
                <div style="margin-block: 30px;text-align: center;">
                    <el-button 
                        type="primary" 
                        @click="record" 
                        style="background: linear-gradient(135deg, #FF7F50 0%, #FF6347 100%);border: none;padding: 12px 40px;border-radius: 12px;font-size: 18px;font-weight: 600;box-shadow: 0 4px 15px rgba(255, 127, 80, 0.3);"
                        class="record-btn"
                    >
                        立即记录
                    </el-button>
                </div>
            </div>
        </div>
        
        <!-- 对话框 -->
        <el-dialog 
            :show-close="false" 
            :visible.sync="dialogGourmetOperaion" 
            width="480px"
            :before-close="cannel"
            class="diet-dialog"
        >
            <div style="padding: 20px 30px;">
                <h3 style="font-size: 20px;color: #5C3D2E;margin-bottom: 20px;text-align: center;">添加菜单</h3>
                
                <div style="margin-bottom: 20px;">
                    <p style="color: #5C3D2E;font-weight: 600;margin-bottom: 8px;">选择菜单</p>
                    <el-select 
                        style="width: 100%;" 
                        size="medium" 
                        v-model="dieHistory.cookbookId" 
                        placeholder="请选择菜单"
                        class="diet-select"
                    >
                        <el-option 
                            v-for="item in cookbooks" 
                            :key="item.id" 
                            :label="item.name" 
                            :value="item.id"
                        >
                        </el-option>
                    </el-select>
                </div>
                
                <div style="margin-bottom: 20px;">
                    <p style="color: #5C3D2E;font-weight: 600;margin-bottom: 8px;">备注</p>
                    <el-input 
                        type="textarea" 
                        :rows="3" 
                        placeholder="请输入备注信息（如用餐时间、特殊制作等）" 
                        v-model="dieHistory.detail"
                        class="diet-textarea"
                    >
                    </el-input>
                </div>
                
                <div style="margin-bottom: 20px;">
                    <p style="color: #5C3D2E;font-weight: 600;margin-bottom: 8px;">食用量（g为单位）</p>
                    <el-input
                        v-model="dieHistory.value"
                        placeholder="请输入食用量"
                        class="diet-input"
                    >
                        <template slot="append">克</template>
                    </el-input>
                </div>
            </div>
            
            <template slot="footer">
                <div style="padding: 0 30px 20px;">
                    <div style="display: flex;justify-content: center;gap: 20px;">
                        <span class="cannel-button" @click="cannel()">
                            取消
                        </span>
                        <span class="confirm-button" @click="addOperation()">
                            确认添加
                        </span>
                    </div>
                </div>
            </template>
        </el-dialog>
    </div>
</template>

<script>
export default {
    name: "DietHistory",
    data() {
        return {
            dieHistory: {},
            dialogGourmetOperaion: false,
            cookbooks: [
                { id: 1, name: '风味腊肠' },
                { id: 2, name: '干香蛙' },
                { id: 3, name: '白炒河粉' },
                { id: 4, name: '红烧狮子头' },
                { id: 5, name: '清蒸鲈鱼' }
            ],
            // 修改：移除模拟数据中的两个菜单
            dietHitoryList: [],
        }
    },
    created() {
        this.fetchCookbookList();
    },
    methods: {
        record() {
            this.$axios.post(`/dietHistory/save`, this.dietHitoryList).then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.$notify({
                        duration: 1000,
                        title: '记录饮食',
                        message: '成功',
                        type: 'success'
                    });
                    this.dietHitoryList = [];
                }
            }).catch(error => {
                console.log("新增饮食信息异常：", error);
            })
        },
        removeCookbook(dieHistory) {
            this.dietHitoryList = this.dietHitoryList.filter(entity => entity.cookbookId !== dieHistory.cookbookId);
        },
        getCookbookName(dieHistory) {
            const cookbook = this.cookbooks.find(entity => entity.id === dieHistory.cookbookId);
            return cookbook ? cookbook.name : '未知菜单';
        },
        fetchCookbookList() {
            this.$axios.get(`/cookbook/querySelectedItemsAll`).then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.cookbooks = data.data;
                }
            }).catch(error => {
                console.log("查询食谱信息异常：", error);
            })
        },
        addOperation() {
            if (!this.dieHistory.cookbookId) {
                this.$message.warning('请选择菜单');
                return;
            }
            this.dietHitoryList.push({...this.dieHistory});
            this.cannel();
        },
        cannel() {
            this.dieHistory = {};
            this.dialogGourmetOperaion = false;
        },
        recordGourmet() {
            this.dialogGourmetOperaion = true;
        },
    }
};
</script>

<style scoped lang="scss">
.record {
    display: flex;
    justify-content: center;
    align-items: center;
    background: white;
    min-height: 200px;
    cursor: pointer;
    border-radius: 16px;
    box-shadow: 0 4px 20px rgba(194, 145, 89, 0.1);
    border: 2px dashed #E8D4B9;
    transition: all 0.3s ease;
    padding: 40px 20px;
    
    &:hover {
        transform: translateY(-5px);
        box-shadow: 0 8px 30px rgba(194, 145, 89, 0.2);
        border-color: #C28B59;
        background: linear-gradient(135deg, #FFF9F0 0%, #FFE9C8 100%);
    }
}

.item-diet {
    padding: 20px;
    border-radius: 12px;
    background: white;
    margin-bottom: 16px;
    box-shadow: 0 2px 8px rgba(194, 145, 89, 0.1);
    border: 1px solid #E8D4B9;
    transition: all 0.3s ease;
    
    &:hover {
        box-shadow: 0 4px 20px rgba(194, 145, 89, 0.15);
        transform: translateY(-2px);
        border-color: #C28B59;
    }
    
    .remove-icon:hover {
        color: #FF4500;
        transform: scale(1.1);
    }
}

.record-btn {
    transition: all 0.3s ease;
    
    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 8px 25px rgba(255, 99, 71, 0.4);
        background: linear-gradient(135deg, #FF6347 0%, #FF4500 100%);
    }
}

// 对话框样式
:deep(.diet-dialog) {
    .el-dialog {
        border-radius: 16px;
        overflow: hidden;
        border: 1px solid #E8D4B9;
        box-shadow: 0 10px 40px rgba(194, 145, 89, 0.3);
    }
    
    .el-dialog__header {
        display: none;
    }
    
    .el-dialog__body {
        padding: 0;
    }
}

.diet-select {
    :deep(.el-input__inner) {
        border: 2px solid #E8D4B9;
        background-color: #FFF9F0;
        color: #5C3D2E;
        border-radius: 10px;
        height: 48px;
        line-height: 48px;
        font-size: 16px;
        padding: 0 15px;
        
        &:focus {
            border-color: #C28B59;
            box-shadow: 0 0 0 2px rgba(194, 145, 89, 0.1);
        }
        
        &::placeholder {
            color: #B8A28C;
        }
    }
    
    :deep(.el-input__suffix) {
        .el-icon-arrow-up {
            color: #C28B59;
        }
    }
}

.diet-textarea {
    :deep(.el-textarea__inner) {
        border: 2px solid #E8D4B9;
        background-color: #FFF9F0;
        color: #5C3D2E;
        border-radius: 10px;
        font-size: 16px;
        padding: 12px 15px;
        line-height: 1.5;
        resize: vertical;
        
        &:focus {
            border-color: #C28B59;
            box-shadow: 0 0 0 2px rgba(194, 145, 89, 0.1);
        }
        
        &::placeholder {
            color: #B8A28C;
        }
    }
}

.diet-input {
    :deep(.el-input-group) {
        .el-input__inner {
            border: 2px solid #E8D4B9;
            border-right: none;
            background-color: #FFF9F0;
            color: #5C3D2E;
            border-radius: 10px 0 0 10px;
            height: 48px;
            line-height: 48px;
            font-size: 16px;
            padding: 0 15px;
            
            &:focus {
                border-color: #C28B59;
                box-shadow: 0 0 0 2px rgba(194, 145, 89, 0.1);
            }
            
            &::placeholder {
                color: #B8A28C;
            }
        }
        
        .el-input-group__append {
            border: 2px solid #E8D4B9;
            border-left: none;
            background-color: #FFF9F0;
            color: #8C6B4A;
            border-radius: 0 10px 10px 0;
            font-size: 16px;
            padding: 0 20px;
        }
    }
}

.cannel-button {
    display: inline-block;
    text-align: center;
    border-radius: 10px;
    height: 48px;
    line-height: 48px;
    width: 140px;
    background: #FFF9F0;
    font-size: 16px;
    font-weight: 600;
    border: 2px solid #E8D4B9;
    color: #8C6B4A;
    padding: 0;
    cursor: pointer;
    user-select: none;
    transition: all 0.3s ease;
    
    &:hover {
        background: #FFF5E6;
        border-color: #C28B59;
        color: #5C3D2E;
        transform: translateY(-2px);
    }
}

.confirm-button {
    display: inline-block;
    text-align: center;
    border-radius: 10px;
    height: 48px;
    line-height: 48px;
    width: 140px;
    background: linear-gradient(135deg, #FF7F50 0%, #FF6347 100%);
    font-size: 16px;
    font-weight: 600;
    border: none;
    color: white;
    padding: 0;
    cursor: pointer;
    user-select: none;
    transition: all 0.3s ease;
    box-shadow: 0 4px 12px rgba(255, 127, 80, 0.3);
    
    &:hover {
        background: linear-gradient(135deg, #FF6347 0%, #FF4500 100%);
        transform: translateY(-2px);
        box-shadow: 0 6px 20px rgba(255, 99, 71, 0.4);
    }
}
</style>