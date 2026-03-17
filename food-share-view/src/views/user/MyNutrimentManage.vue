<template>
    <el-row style="background-color: #fffaf0;padding: 5px 0;border-radius: 5px;">
        <el-row style="padding: 10px;margin-left: 5px;">
            <el-row style="display: flex;justify-content: left;gap: 6px;">
                <span class="edit-button" @click="add()">
                    配置营养素
                </span>
                <el-date-picker style="width: 216px;" @change="fetchFreshData" size="small" v-model="searchTime"
                    type="daterange" range-separator="至" start-placeholder="创建开始" end-placeholder="创建结束">
                </el-date-picker>
                <el-input size="small" style="width: 166px;" v-model="nutrimentQueryDto.name" placeholder="营养素名"
                    clearable @clear="handleFilterClear">
                    <el-button slot="append" @click="handleFilter" icon="el-icon-search"></el-button>
                </el-input>
            </el-row>
        </el-row>
        <el-row style="margin: 0 22px;border-top: 1px solid #f5e6d3;">
            <el-table :stripe="true" :data="tableData" style="width: 100%"
                :header-cell-style="{backgroundColor: '#fef7e0', color: '#8b4513'}">
                <el-table-column prop="name" label="营养素名"></el-table-column>
                <el-table-column prop="detail" label="描述" width="468"></el-table-column>
                <el-table-column prop="unit" label="单位" width="108"></el-table-column>
                <el-table-column prop="isPublish" label="是否公开" width="118">
                    <template slot-scope="scope">
                        <span>{{ scope.row.isPublish ? '公开' : '私人' }}</span>
                    </template>
                </el-table-column>
                <el-table-column prop="createTime" label="创建时间" :sortable="true" width="168"></el-table-column>
                <el-table-column label="操作" width="110">
                    <template slot-scope="scope">
                        <span class="text-button" @click="handleEdit(scope.row)">修改</span>
                        <span class="text-button" @click="handleDelete(scope.row)">删除</span>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination style="margin:10px 0;float: right;" @size-change="handleSizeChange"
                @current-change="handleCurrentChange" :current-page="currentPage" :page-sizes="[20, 50]"
                :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper"
                :total="totalItems"></el-pagination>
        </el-row>
        <el-dialog :show-close="false" :visible.sync="dialogNutrimentOperaion" width="25%"
            custom-class="warm-yellow-dialog">
            <div style="padding:16px 20px;">
                <p class="dialog-title">{{ isOperation ? '营养素修改' : '营养素新增' }}</p>
                <div style="margin-bottom: 15px;">
                    <span class="dialog-subtitle">*单位</span>
                    <input class="dialog-input" v-model="data.name" placeholder="营养素名称" />
                </div>
                <div style="margin-bottom: 15px;">
                    <span class="dialog-subtitle">*单位</span>
                    <input class="dialog-input" v-model="data.unit" placeholder="单位" />
                </div>
                <div>
                    <span class="dialog-subtitle">*描述</span>
                    <el-input type="textarea" :rows="3" placeholder="用一句话描述这项配置吧" v-model="data.detail">
                    </el-input>
                </div>
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
            tableData: [],
            delectedRows: [],
            searchTime: [],
            dialogNutrimentOperaion: false,
            isOperation: false,
            nutrimentQueryDto: {}, // 搜索条件
            publishStatus: [{ value: null, label: '全部' }, { value: 0, label: '私人' }, { value: 1, label: '公开' }],
        };
    },
    created() {
        this.fetchFreshData();
    },
    methods: {
        addOperation() {
            this.$axios.post('/nutriment/save', this.data).then(res => {
                if (res.data.code === 200) {
                    this.$notify({
                        duration: 1000,
                        title: '营养素操作',
                        message: '新增成功',
                        type: 'success'
                    });
                    this.cannel();
                    this.fetchFreshData();
                }
            }).catch(error => {
                console.log("新增营养素异常：", error);
            });
        },
        updateOperation() {
            this.$axios.put('/nutriment/update', this.data).then(res => {
                if (res.data.code === 200) {
                    this.$notify({
                        duration: 1000,
                        title: '营养素操作',
                        message: '修改成功',
                        type: 'success'
                    });
                    this.cannel();
                    this.fetchFreshData();
                }
            }).catch(error => {
                console.log("修改营养素异常：", error);
            });
        },
        cannel() {
            this.dialogNutrimentOperaion = false;
            this.isOperation = false;
            this.data = {};
        },
        add() {
            this.dialogNutrimentOperaion = true;
        },
        // 批量删除数据
        async batchDelete() {
            if (!this.delectedRows.length) {
                this.$message(`未选中任何数据`);
                return;
            }
            const confirmed = await this.$swalConfirm({
                title: '删除营养素数据',
                text: `删除后不可恢复，是否继续？`,
                icon: 'warning',
            });
            if (confirmed) {
                try {
                    let ids = this.delectedRows.map(entity => entity.id);
                    console.log("删除对象：" ,ids);
                    
                    const response = await this.$axios.post(`/nutriment/batchDelete`, ids);
                    if (response.data.code === 200) {
                        this.$notify({
                            duration: 1000,
                            title: '信息删除',
                            message: '删除成功',
                            type: 'success'
                        });
                        this.fetchFreshData();
                        this.delectedRows = [];
                        return;
                    }
                } catch (error) {
                    this.$message.error("营养素删除异常：", error);
                    console.error(`营养素删除异常：`, error);
                }
            }
        },
        // 信息查询
        async fetchFreshData() {
            try {
                this.tableData = [];
                let startTime = null;
                let endTime = null;
                if (this.searchTime != null && this.searchTime.length === 2) {
                    const [startDate, endDate] = await Promise.all(this.searchTime.map(date => date.toISOString()));
                    startTime = `${startDate.split('T')[0]}T00:00:00`;
                    endTime = `${endDate.split('T')[0]}T23:59:59`;
                }
                // 请求参数
                const params = {
                    current: this.currentPage,
                    size: this.pageSize,
                    key: this.filterText,
                    startTime: startTime,
                    endTime: endTime,
                    ...this.nutrimentQueryDto
                };
                const response = await this.$axios.post('/nutriment/queryUser', params);
                const { data } = response;
                this.tableData = data.data;
                this.totalItems = data.total;
            } catch (error) {
                this.$message.error("查询营养素异常:", error);
                console.error('查询营养素异常:', error);
            }
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
        handleEdit(row) {
            this.data = row;
            this.dialogNutrimentOperaion = true;
            this.isOperation = true;
        },
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

/* 分页样式 */
:deep(.el-pagination) {
    .btn-prev, .btn-next, .number, .el-icon {
        color: #8b4513;
        
        &:hover {
            color: #d2691e;
        }
    }
    
    .active {
        background-color: #f0c674;
        color: #8b4513;
        border-color: #f0c674;
    }
    
    .el-pagination__total {
        color: #8b4513;
    }
    
    .el-input__inner {
        background-color: #fffaf0;
        border-color: #f5e6d3;
        color: #8b4513;
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
    
    .el-input-group__append {
        background-color: #f0c674;
        border-color: #e8b85c;
        color: #8b4513;
        
        .el-button {
            color: #8b4513;
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

/* 文本域样式 */
:deep(.el-textarea) {
    .el-textarea__inner {
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

/* 日期选择器样式 */
:deep(.el-date-editor) {
    .el-input__inner {
        background-color: #fffaf0;
        border-color: #f5e6d3;
        color: #8b4513;
    }
    
    .el-range-separator {
        color: #8b4513;
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