<template>
    <el-row style="background-color: #fffaf0;padding: 5px 0;border-radius: 5px;">
        <el-row style="padding: 10px;margin-left: 5px;">
            <el-row style="display: flex;justify-content: left;gap: 6px;">
                <span class="edit-button" @click="add()">
                    新增食谱配置
                </span>
            </el-row>
        </el-row>
        <el-row style="margin: 0 22px;border-top: 1px solid #f5e6d3;">
            <el-table :stripe="true" :data="tableData"
                :header-cell-style="{backgroundColor: '#fef7e0', color: '#8b4513'}">
                <el-table-column prop="cookbookId" :sortable="true" width="120" label="食谱ID"></el-table-column>
                <el-table-column prop="nutrimentId" :sortable="true" width="120" label="营养素ID"></el-table-column>
                <el-table-column prop="cookbookName" label="食谱"></el-table-column>
                <el-table-column prop="nutrimentName" width="120" label="营养项"></el-table-column>
                <el-table-column prop="nutrimentUnit" width="130" label="营养素单位"></el-table-column>
                <el-table-column prop="g100Value" width="170" label="每100g中含量"></el-table-column>
                <el-table-column label="操作" width="110">
                    <template slot-scope="scope">
                        <span class="text-button" @click="handleEdit(scope.row)">编辑</span>
                        <span class="text-button" @click="handleDelete(scope.row)">删除</span>
                    </template>
                </el-table-column>
            </el-table>
        </el-row>
        <!-- 操作面板 -->
        <el-dialog :show-close="false" :visible.sync="dialogCategoryOperaion" width="18%"
            custom-class="warm-yellow-dialog">
            <div style="padding:16px 20px;">
                <p class="dialog-title">{{ isOperation ? '修改营养素信息' : '配置营养素' }}</p>
                <el-row style="margin-bottom: 15px;">
                    <span class="dialog-subtitle">选食谱</span>
                    <el-select style="width: 160px;" @change="fetchFreshData" size="small" v-model="data.cookbookId"
                        placeholder="食谱">
                        <el-option v-for="item in cookbookItems" :key="item.id" :label="item.name" :value="item.id">
                        </el-option>
                    </el-select>
                </el-row>
                <el-row style="margin-bottom: 15px;">
                    <span class="dialog-subtitle">营养素</span>
                    <el-select style="width: 160px;" @change="fetchFreshData" size="small" v-model="data.nutrimentId"
                        placeholder="营养素">
                        <el-option v-for="item in nutrimentItems" :key="item.id" :label="item.name" :value="item.id">
                        </el-option>
                    </el-select>
                </el-row>
                <el-row>
                    <span class="dialog-subtitle">每100g中值含量</span>
                    <input class="dialog-input" v-model="data.g100Value" placeholder="输入" />
                </el-row>
            </div>
            <span slot="footer" class="dialog-footer" style="margin-top: 10px;">
                <span class="channel-button" @click="cannel()">
                    取消操作
                </span>
                <span v-if="!isOperation" class="edit-button" @click="addOperation()">
                    确定新增
                </span>
                <span v-else class="edit-button" @click="updateOperation()">
                    确定修改
                </span>
            </span>
        </el-dialog>
    </el-row>
</template>

<script>
export default {
    data() {
        return {
            data: {},
            filterText: '',
            currentPage: 1,
            pageSize: 20,
            totalItems: 0,
            dialogCategoryOperaion: false, // 开关
            isOperation: false, // 默认新增
            tableData: [],
            delectedRows: [],
            cookbookNutrimentQueryDto: {}, // 搜索条件
            cookbookItems: [],
            nutrimentItems: []
        };
    },
    created() {
        this.fetchCookBookListItem();
        this.fetchNutrimentListItem();
        this.fetchFreshData();
    },
    methods: {
        // 食谱下拉选择器数据
        async fetchCookBookListItem() {
            try {
                const response = await this.$axios.get(`/cookbook/querySelectedItemsUser`);
                if (response.data.code === 200) {
                    this.cookbookItems = response.data.data;
                    
                }
            } catch (error) {
                console.log(error);
            }
        },
        // 营养素下拉选择器数据
        async fetchNutrimentListItem() {
            try {
                const response = await this.$axios.get(`/nutriment/querySelectedItemsUser`);
                if (response.data.code === 200) {
                    this.nutrimentItems = response.data.data;
                }
            } catch (error) {
                console.log(error);
            }
        },
        cannel() {
            this.data = {};
            this.dialogCategoryOperaion = false;
            this.isOperation = false;
        },
        // 批量删除数据
        async batchDelete() {
            if (!this.delectedRows.length) {
                this.$message(`未选中任何数据`);
                return;
            }
            const confirmed = await this.$swalConfirm({
                title: '删除食谱营养素关联数据',
                text: `删除后不可恢复，是否继续？`,
                icon: 'warning',
            });
            if (confirmed) {
                try {
                    let ids = this.delectedRows.map(entity => entity.id);
                    const response = await this.$axios.post(`/cookbookNutriment/batchDelete`, ids);
                    if (response.data.code === 200) {
                        this.$notify({
                            duration: 1000,
                            title: '信息删除',
                            message: '删除成功',
                            type: 'success'
                        });
                        this.fetchFreshData();
                        return;
                    }
                } catch (error) {
                    this.$message.error("食谱营养素关联信息删除异常：", error);
                    console.error(`食谱营养素关联信息删除异常：`, error);
                }
            }
        },
        // 修改信息
        async updateOperation() {
            try {
                const response = await this.$axios.put('/cookbookNutriment/update', this.data);
                if (response.data.code === 200) {
                    this.$notify({
                        duration: 1000,
                        title: '信息修改',
                        message: '修改成功',
                        type: 'success'
                    });
                    this.cannel();
                    this.fetchFreshData();
                }
            } catch (error) {
                console.error('修改出错:', error);
            }
        },
        // 信息新增
        async addOperation() {
            try {
                const response = await this.$axios.post('/cookbookNutriment/save', this.data);
                if (response.data.code === 200) {
                    this.$notify({
                        duration: 1000,
                        title: '配置新增',
                        message: '新增成功',
                        type: 'success'
                    });
                    this.cannel();
                    this.fetchFreshData();
                } else {
                    this.$notify({
                        duration: 1000,
                        title: '配置新增',
                        message: response.data.msg,
                        type: 'error'
                    });
                }
            } catch (error) {
                console.error('信息新增出错:', error);
                this.$message.error('提交失败，请稍后再试！');
            }
        },
        // 信息查询
        async fetchFreshData() {
            try {
                const response = await this.$axios.post('/cookbookNutriment/queryUser');
                const { data } = response;
                this.tableData = data.data;
            } catch (error) {
                this.$message.error("查询食谱营养素关联信息异常:", error);
                console.error('查询食谱营养素关联信息异常:', error);
            }
        },
        add() {
            this.dialogCategoryOperaion = true;
        },
        handleFilter() {
            this.currentPage = 1;
            this.fetchFreshData();
        },
        handleFilterClear() {
            this.filterText = '';
            this.handleFilter();
        },
        handleSizeChange(val) {
            this.pageSize = val;
            this.currentPage = 1;
            this.fetchFreshData();
        },
        handleCurrentChange(val) {
            this.currentPage = val;
            this.fetchFreshData();
        },
        // 食谱营养素关联修改按钮点击事件 
        handleEdit(row) {
            this.dialogCategoryOperaion = true;
            this.isOperation = true;
            this.data = { ...row }
        },
        // 食谱营养素关联修改按钮删除事件 
        handleDelete(row) {
            this.delectedRows = [row];
            this.batchDelete();
        }
    },
};
</script>
<style scoped lang="scss">
/* 基础背景颜色 - 暖黄色系 */
:deep(.warm-yellow-dialog) {
    background-color: #fffaf0 !important;
    border: 1px solid #f5e6d3 !important;
    border-radius: 8px !important;
    
    .el-dialog__header {
        background-color: #fef7e0 !important;
        border-bottom: 1px solid #f5e6d3 !important;
    }
    
    .el-dialog__body {
        background-color: #fffaf0 !important;
    }
    
    .el-dialog__footer {
        background-color: #fef7e0 !important;
        border-top: 1px solid #f5e6d3 !important;
    }
}

/* 表格样式优化 */
:deep(.el-table) {
    background-color: #fffaf0;
    color: #8b4513;
    
    th {
        background-color: #fef7e0 !important;
        color: #8b4513;
        font-weight: 600;
    }
    
    tr {
        background-color: #fffaf0;
        
        &:hover {
            background-color: #fef7e0;
        }
    }
    
    .el-table__row {
        &:nth-child(2n) {
            background-color: #fef7e0;
        }
        
        &:nth-child(2n+1) {
            background-color: #fffaf0;
        }
    }
}

/* 弹窗样式 */
.dialog-title {
    color: #8b4513;
    font-size: 18px;
    font-weight: bold;
    margin-bottom: 15px;
}

.dialog-subtitle {
    display: block;
    color: #a0522d;
    font-size: 14px;
    font-weight: 600;
    margin-bottom: 5px;
}

/* 输入框样式 */
:deep(.el-input) {
    .el-input__inner {
        background-color: #fffaf0;
        border-color: #f5e6d3;
        color: #8b4513;
        
        &:focus {
            border-color: #f0c674;
        }
        
        &::placeholder {
            color: #c9a880;
        }
    }
}

.dialog-input {
    width: 100%;
    padding: 8px 12px;
    border: 1px solid #f5e6d3;
    border-radius: 4px;
    background-color: #fffaf0;
    color: #8b4513;
    margin-bottom: 10px;
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

/* 选择框样式 */
:deep(.el-select) {
    .el-input__inner {
        background-color: #fffaf0;
        border-color: #f5e6d3;
        color: #8b4513;
    }
    
    .el-select-dropdown {
        background-color: #fffaf0;
        border: 1px solid #f5e6d3;
    }
    
    .el-select-dropdown__item {
        color: #8b4513;
        
        &:hover, &.selected {
            background-color: #fef7e0;
        }
    }
}

/* 按钮样式 */
.edit-button {
    display: inline-block;
    padding: 6px 12px;
    background-color: #f0c674;
    color: #8b4513;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s ease;
    border: 1px solid #e8b85c;
    font-size: 12px;
    
    &:hover {
        background-color: #e8b85c;
        color: #fff;
    }
}

.channel-button {
    display: inline-block;
    padding: 6px 12px;
    background-color: #fffaf0;
    color: #8b4513;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s ease;
    border: 1px solid #f5e6d3;
    font-size: 12px;
    
    &:hover {
        background-color: #f5e6d3;
    }
}

.text-button {
    color: #d2691e;
    cursor: pointer;
    margin-right: 10px;
    transition: color 0.3s ease;
    
    &:hover {
        color: #8b4513;
        text-decoration: underline;
    }
}
</style>