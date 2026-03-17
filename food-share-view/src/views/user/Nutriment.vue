<template>
    <div style="background-color: #FFF9F0;padding: 20px;min-height: 100vh;">
        <el-row style="background-color: #FFFFFF;padding: 20px;border-radius: 12px;box-shadow: 0 4px 20px rgba(194, 145, 89, 0.1);border: 1px solid #E8D4B9;">
            <el-row style="padding: 10px;margin-left: 5px;">
                <el-row>
                    <span style="font-size: 28px;font-weight: bold;display: inline-block;padding: 0 10px;color: #5C3D2E;">我的饮食</span>
                    <span style="float: right;">
                        <el-date-picker 
                            style="width: 216px;margin-right: 5px;" 
                            @change="fetchFreshData" 
                            size="small" 
                            v-model="searchTime"
                            type="daterange" 
                            range-separator="至" 
                            start-placeholder="记录开始" 
                            end-placeholder="记录结束"
                            class="date-picker"
                        >
                        </el-date-picker>
                        <el-input 
                            size="small" 
                            style="width: 166px;" 
                            v-model="dietHistoryQueryDto.cookbookId"
                            placeholder="食谱编号" 
                            clearable 
                            @clear="handleFilterClear"
                            class="search-input"
                        >
                            <el-button 
                                slot="append" 
                                @click="handleFilter" 
                                icon="el-icon-search"
                                style="background-color: #C28B59;border-color: #C28B59;color: white;"
                            ></el-button>
                        </el-input>
                    </span>
                </el-row>
            </el-row>
            <el-row style="margin: 0 22px;border-top: 2px solid #E8D4B9;padding-top: 20px;">
                <el-table 
                    :stripe="true" 
                    :data="tableData" 
                    style="width: 100%"
                    class="diet-table"
                >
                    <el-table-column prop="cookbookCover" width="80" label="封面">
                        <template slot-scope="scope">
                            <img 
                                :src="scope.row.cookbookCover" 
                                style="width: 50px;height: 50px;border-radius: 8px;border: 2px solid #FFE9C8;object-fit: cover;" 
                                alt=""
                                srcset=""
                            >
                        </template>
                    </el-table-column>
                    <el-table-column prop="cookbookTitle" label="食谱" width="180">
                        <template slot-scope="scope">
                            <span style="font-weight: 600;color: #5C3D2E;">{{ scope.row.cookbookTitle }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="detail" label="备注" width="200">
                        <template slot-scope="scope">
                            <span style="color: #8C6B4A;">{{ scope.row.detail || '暂无备注' }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="value" label="食量" width="120">
                        <template slot-scope="scope">
                            <span style="display: inline-block;background-color: #FFF5E6;padding: 6px 12px;border-radius: 12px;color: #A8723A;font-weight: 600;border: 1px solid #E8D4B9;">
                                {{ scope.row.value }} g
                            </span>
                        </template>
                    </el-table-column>
                    <el-table-column prop="createTime" label="记录时间" width="168">
                        <template slot-scope="scope">
                            <span style="color: #8C6B4A;">{{ scope.row.createTime }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column label="操作" width="140">
                        <template slot-scope="scope">
                            <span class="edit-button" @click="handleEdit(scope.row)">编辑</span>
                            <span class="delete-button" @click="handleDelete(scope.row)">删除</span>
                        </template>
                    </el-table-column>
                </el-table>
                <el-pagination 
                    style="margin:20px 0;float: right;" 
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange" 
                    :current-page="currentPage" 
                    :page-sizes="[20, 50]"
                    :page-size="pageSize" 
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="totalItems"
                    class="diet-pagination"
                ></el-pagination>
            </el-row>
            <div style="padding-inline: 20px;margin-top: 30px;">
                <el-row style="margin-bottom: 20px;">
                    <span class="dialog-hover" style="font-weight: 600;color: #5C3D2E;margin-right: 10px;">营养素</span>
                    <el-select 
                        style="width: 160px;" 
                        @change="loadNutrimentData" 
                        size="small"
                        v-model="dietQueryDto.nutrimentId" 
                        placeholder="营养素"
                        class="nutrient-select"
                    >
                        <el-option 
                            v-for="item in nutrimentItems" 
                            :key="item.id" 
                            :label="item.name" 
                            :value="item.id"
                        >
                        </el-option>
                    </el-select>
                </el-row>
                <div style="background-color: #FFF9F0;border-radius: 12px;padding: 20px;border: 1px solid #E8D4B9;margin-bottom: 20px;">
                    <LineChart 
                        tag="营养摄入情况" 
                        height="400px" 
                        @on-selected="onSelected" 
                        :values="values" 
                        :date="dates" 
                    />
                </div>
            </div>
        </el-row>
        
        <!-- 编辑对话框 -->
        <el-dialog 
            :show-close="false" 
            :visible.sync="dialogDietOperaion" 
            width="480px"
            :before-close="cannel"
            class="diet-edit-dialog"
        >
            <div style="padding: 20px 30px;">
                <h3 style="font-size: 20px;color: #5C3D2E;margin-bottom: 20px;text-align: center;">编辑饮食记录</h3>
                
                <div style="margin-bottom: 20px;">
                    <p style="color: #5C3D2E;font-weight: 600;margin-bottom: 8px;">选择菜单</p>
                    <el-select 
                        style="width: 100%;" 
                        size="small" 
                        v-model="data.cookbookId" 
                        placeholder="选择菜单"
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
                        placeholder="请输入备注信息" 
                        v-model="data.detail"
                        class="diet-textarea"
                    >
                    </el-input>
                </div>
                
                <div style="margin-bottom: 20px;">
                    <p style="color: #5C3D2E;font-weight: 600;margin-bottom: 8px;">食用量（g为单位）</p>
                    <el-input
                        v-model="data.value"
                        placeholder="请输入食用量"
                        class="diet-input"
                    >
                        <template slot="append">克</template>
                    </el-input>
                </div>
            </div>
            <div slot="footer" style="padding: 0 30px 20px;">
                <div style="display: flex;justify-content: center;gap: 20px;">
                    <span class="cannel-button" @click="cannel()">
                        取消
                    </span>
                    <span class="confirm-button" @click="updateOperation()">
                        确认修改
                    </span>
                </div>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import LineChart from "@/components/LineChart"
export default {
    components: { LineChart },
    data() {
        return {
            data: {},
            filterText: '',
            currentPage: 1,
            pageSize: 20,
            totalItems: 0,
            dietQueryDto: { day: 3650 },
            dialogDietOperaion: false, // 开关
            tableData: [
                {
                    id: 1,
                    cookbookCover: 'https://images.unsplash.com/photo-1565299624946-b28f40a0ae38',
                    cookbookTitle: '风味腊肠',
                    detail: '午餐，加了一点辣椒',
                    value: 250,
                    createTime: '2024-12-04 12:30:00',
                    cookbookId: 1
                },
                {
                    id: 2,
                    cookbookCover: 'https://images.unsplash.com/photo-1565299507177-b0ac66763828',
                    cookbookTitle: '白炒河粉',
                    detail: '早餐，加了两个鸡蛋',
                    value: 300,
                    createTime: '2024-12-04 08:15:00',
                    cookbookId: 3
                }
            ],
            delectedRows: [],
            cookbooks: [
                { id: 1, name: '风味腊肠' },
                { id: 2, name: '干香蛙' },
                { id: 3, name: '白炒河粉' },
                { id: 4, name: '红烧狮子头' },
                { id: 5, name: '清蒸鲈鱼' }
            ],
            searchTime: [],
            dietHistoryQueryDto: {}, // 搜索条件
            nutrimentUseList: [], // 营养素摄入情况
            values: [25, 30, 28, 35, 32, 40],
            dates: ['12-01', '12-02', '12-03', '12-04', '12-05', '12-06'],
            nutrimentItems: [
                { id: 1, name: '蛋白质' },
                { id: 2, name: '碳水化合物' },
                { id: 3, name: '脂肪' },
                { id: 4, name: '纤维素' }
            ]
        };
    },
    created() {
        this.fetchFreshData();
        this.fetchCookbookList();
        this.loadNutrimentData();
        this.fetchNutrimentListItem();
    },
    methods: {
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
        onSelected(day) {
            this.dietQueryDto.day = day;
            this.loadNutrimentData();
        },
        updateOperation() {
            this.$axios.put(`/dietHistory/update`, this.data).then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.$notify({
                        duration: 1000,
                        title: '饮食操作',
                        message: '修改成功',
                        type: 'success'
                    });
                    this.cannel();
                    this.fetchFreshData();
                }
            }).catch(error => {
                console.log("查询食谱信息异常：", error);
            })
        },
        // 查询营养摄入情况
        loadNutrimentData() {
            this.$axios.post(`/dietHistory/queryDietNutrimentInfo`, this.dietQueryDto).then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.nutrimentUseList = data.data;
                    this.dates = this.nutrimentUseList.map(entity => entity.time);
                    this.values = this.nutrimentUseList.map(entity => entity.value);
                }
            }).catch(error => {
                console.log("查询营养素摄入信息异常：", error);
            })
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
        cannel() {
            this.data = {};
            this.dialogDietOperaion = false;
            this.isOperation = false;
        },
        // 批量删除数据
        async batchDelete() {
            if (!this.delectedRows.length) {
                this.$message(`未选中任何数据`);
                return;
            }
            const confirmed = await this.$swalConfirm({
                title: '删除用户饮食数据',
                text: `删除后不可恢复，是否继续？`,
                icon: 'warning',
            });
            if (confirmed) {
                try {
                    let ids = this.delectedRows.map(entity => entity.id);
                    const response = await this.$axios.post(`/dietHistory/batchDelete`, ids);
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
                    this.$message.error("用户饮食信息删除异常：", error);
                    console.error(`用户饮食信息删除异常：`, error);
                }
            }
        },
        // 信息新增
        async addOperation() {
            try {
                const response = await this.$axios.post('/dietHistory/save', this.data);
                if (response.data.code === 200) {
                    this.$notify({
                        duration: 1000,
                        title: '信息新增',
                        message: '新增成功',
                        type: 'success'
                    });
                    this.cannel();
                    this.fetchFreshData();
                }
            } catch (error) {
                console.error('信息新增出错:', error);
                this.$message.error('提交失败，请稍后再试！');
            }
        },
        // 信息查询
        async fetchFreshData() {
            try {
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
                    ...this.dietHistoryQueryDto
                };
                const response = await this.$axios.post('/dietHistory/queryUser', params);
                const { data } = response;
                this.tableData = data.data;
                this.totalItems = data.total;
            } catch (error) {
                this.$message.error("查询用户饮食信息异常:", error);
                console.error('查询用户饮食信息异常:', error);
            }
        },
        add() {
            this.dialogDietOperaion = true;
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
        // 用户饮食修改按钮点击事件 
        handleEdit(row) {
            this.dialogDietOperaion = true;
            this.data = { ...row }
        },
        // 用户饮食修改按钮删除事件 
        handleDelete(row) {
            this.delectedRows.push(row);
            this.batchDelete();
        }
    },
};
</script>

<style scoped lang="scss">
// 日期选择器样式
:deep(.date-picker) {
    .el-input__inner {
        border: 2px solid #E8D4B9;
        background-color: #FFF9F0;
        color: #5C3D2E;
        border-radius: 8px;
        height: 32px;
        line-height: 32px;
        
        &:focus {
            border-color: #C28B59;
            box-shadow: 0 0 0 2px rgba(194, 145, 89, 0.1);
        }
    }
    
    .el-range-separator {
        color: #8C6B4A;
    }
}

// 搜索框样式
:deep(.search-input) {
    .el-input__inner {
        border: 2px solid #E8D4B9;
        background-color: #FFF9F0;
        color: #5C3D2E;
        border-radius: 8px 0 0 8px;
        height: 32px;
        line-height: 32px;
        
        &:focus {
            border-color: #C28B59;
            box-shadow: 0 0 0 2px rgba(194, 145, 89, 0.1);
        }
    }
    
    .el-input-group__append {
        border: 2px solid #C28B59;
        border-left: none;
        background-color: #C28B59;
        border-radius: 0 8px 8px 0;
        
        .el-button {
            color: white;
            
            &:hover {
                background-color: #A8723A;
            }
        }
    }
}

// 营养素选择器样式
:deep(.nutrient-select) {
    .el-input__inner {
        border: 2px solid #E8D4B9;
        background-color: #FFF9F0;
        color: #5C3D2E;
        border-radius: 8px;
        height: 32px;
        line-height: 32px;
        
        &:focus {
            border-color: #C28B59;
            box-shadow: 0 0 0 2px rgba(194, 145, 89, 0.1);
        }
    }
}

// 表格样式
:deep(.diet-table) {
    border-radius: 8px;
    overflow: hidden;
    border: 1px solid #E8D4B9;
    
    .el-table__header-wrapper {
        th {
            background-color: #FFF9F0;
            color: #5C3D2E;
            font-weight: 600;
            border-bottom: 2px solid #E8D4B9;
        }
    }
    
    .el-table__body-wrapper {
        tr {
            &:hover {
                td {
                    background-color: #FFF5E6;
                }
            }
            
            td {
                border-bottom: 1px solid #E8D4B9;
                color: #8C6B4A;
            }
        }
    }
}

// 分页样式
:deep(.diet-pagination) {
    .el-pagination__total {
        color: #8C6B4A;
    }
    
    .el-pager {
        li {
            color: #8C6B4A;
            border: 1px solid #E8D4B9;
            margin: 0 2px;
            border-radius: 6px;
            
            &:hover {
                color: #C28B59;
            }
            
            &.active {
                background-color: #C28B59;
                border-color: #C28B59;
                color: white;
            }
        }
    }
    
    .btn-prev, .btn-next {
        color: #8C6B4A;
        border: 1px solid #E8D4B9;
        border-radius: 6px;
        margin: 0 2px;
        
        &:hover {
            color: #C28B59;
        }
        
        &:disabled {
            color: #D4C4B9;
            border-color: #F0E6D9;
        }
    }
}

// 编辑按钮样式
.edit-button {
    display: inline-block;
    padding: 6px 12px;
    background: linear-gradient(135deg, #D4A55E 0%, #C28B59 100%);
    color: white;
    border-radius: 6px;
    font-size: 13px;
    font-weight: 500;
    cursor: pointer;
    margin-right: 8px;
    transition: all 0.3s;
    
    &:hover {
        background: linear-gradient(135deg, #C28B59 0%, #A8723A 100%);
        transform: translateY(-1px);
        box-shadow: 0 4px 8px rgba(194, 145, 89, 0.2);
    }
}

// 删除按钮样式
.delete-button {
    display: inline-block;
    padding: 6px 12px;
    background: linear-gradient(135deg, #FF6B6B 0%, #FF4757 100%);
    color: white;
    border-radius: 6px;
    font-size: 13px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.3s;
    
    &:hover {
        background: linear-gradient(135deg, #FF4757 0%, #FF3838 100%);
        transform: translateY(-1px);
        box-shadow: 0 4px 8px rgba(255, 71, 87, 0.2);
    }
}

// 对话框样式
:deep(.diet-edit-dialog) {
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

// 对话框中的选择器样式
:deep(.diet-select) {
    .el-input__inner {
        border: 2px solid #E8D4B9;
        background-color: #FFF9F0;
        color: #5C3D2E;
        border-radius: 10px;
        height: 40px;
        line-height: 40px;
        font-size: 16px;
        
        &:focus {
            border-color: #C28B59;
            box-shadow: 0 0 0 2px rgba(194, 145, 89, 0.1);
        }
    }
}

// 对话框中的文本区域样式
:deep(.diet-textarea) {
    .el-textarea__inner {
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
    }
}

// 对话框中的输入框样式
:deep(.diet-input) {
    .el-input-group {
        .el-input__inner {
            border: 2px solid #E8D4B9;
            border-right: none;
            background-color: #FFF9F0;
            color: #5C3D2E;
            border-radius: 10px 0 0 10px;
            height: 40px;
            line-height: 40px;
            font-size: 16px;
            
            &:focus {
                border-color: #C28B59;
                box-shadow: 0 0 0 2px rgba(194, 145, 89, 0.1);
            }
        }
        
        .el-input-group__append {
            border: 2px solid #E8D4B9;
            border-left: none;
            background-color: #FFF9F0;
            color: #8C6B4A;
            border-radius: 0 10px 10px 0;
            font-size: 16px;
        }
    }
}

// 取消按钮样式
.cannel-button {
    display: inline-block;
    text-align: center;
    border-radius: 10px;
    height: 40px;
    line-height: 40px;
    width: 120px;
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

// 确认按钮样式
.confirm-button {
    display: inline-block;
    text-align: center;
    border-radius: 10px;
    height: 40px;
    line-height: 40px;
    width: 120px;
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