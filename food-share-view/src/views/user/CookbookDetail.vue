<template>
    <div style="background-color: #FFF9F0;min-height: 100vh;padding: 20px 0;">
        <!-- 返回按钮 -->
        <div style="max-width: 1200px;margin: 0 auto 20px;padding: 0 40px;">
            <button 
                @click="goBack" 
                style="
                    display: inline-flex;
                    align-items: center;
                    gap: 8px;
                    padding: 10px 20px;
                    background-color: white;
                    border: 2px solid #E8D4B9;
                    border-radius: 8px;
                    color: #5C3D2E;
                    font-size: 16px;
                    font-weight: 600;
                    cursor: pointer;
                    transition: all 0.3s ease;
                    box-shadow: 0 2px 8px rgba(194, 145, 89, 0.1);
                "
                class="back-button"
            >
                <i class="el-icon-back" style="font-size: 18px;"></i>
                返回
            </button>
        </div>

        <div class="cookbook-container">
            <!-- 菜谱头部信息 -->
            <div class="cookbook-header">
                <h1 class="cookbook-title">{{ cookbook.title || '食谱详情' }}</h1>
                <div class="cookbook-meta">
                    <span class="meta-item">
                        <i class="el-icon-fork-spoon"></i>
                        所属菜系：{{ cookbook.categoryName || '未分类' }}
                    </span>
                    <span class="meta-item">
                        <i class="el-icon-time"></i>
                        收录于：{{ formatDate(cookbook.createTime) }}
                    </span>
                </div>
            </div>

            <!-- 菜谱主要内容 -->
            <div class="cookbook-content">
                <!-- 方法1: 使用v-html直接渲染HTML表格（兼容原始格式） -->
                <div v-if="cookbook.content && cookbook.content.includes('<table')" 
                     class="content-section html-content">
                    <div v-html="processTableHTML(cookbook.content)"></div>
                </div>

                <!-- 方法2: 使用结构化表格组件 -->
                <div v-else-if="hasStructuredData" class="content-section structured-content">
                    <!-- 材料表格 -->
                    <div v-if="ingredientsData.length > 0" class="section ingredients-section">
                        <h2 class="section-title">
                            <i class="el-icon-shopping-cart-full"></i>
                            所需材料
                        </h2>
                        <div class="table-container">
                            <el-table 
                                :data="ingredientsData" 
                                border
                                stripe
                                class="ingredients-table"
                            >
                                <el-table-column 
                                    prop="category" 
                                    label="类别" 
                                    width="120"
                                />
                                <el-table-column 
                                    prop="ingredient" 
                                    label="食材" 
                                />
                                <el-table-column 
                                    prop="remark" 
                                    label="备注" 
                                />
                            </el-table>
                        </div>
                    </div>

                    <!-- 步骤表格 -->
                    <div v-if="stepsData.length > 0" class="section steps-section">
                        <h2 class="section-title">
                            <i class="el-icon-guide"></i>
                            烹饪步骤
                        </h2>
                        <div class="steps-list">
                            <div v-for="(step, index) in stepsData" :key="index" class="step-item">
                                <div class="step-number">{{ index + 1 }}</div>
                                <div class="step-content">
                                    <div class="step-desc">{{ step.description }}</div>
                                    <div v-if="step.time" class="step-time">⏱️ {{ step.time }}</div>
                                    <div v-if="step.tip" class="step-tip">💡 {{ step.tip }}</div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- 方法3: 显示纯文本内容 -->
                <div v-else-if="cookbook.content" class="content-section text-content">
                    <div class="text-content-wrapper">{{ cookbook.content }}</div>
                </div>

                <!-- 无内容提示 -->
                <div v-else class="empty-content">
                    <i class="el-icon-document"></i>
                    <p>暂无食谱内容</p>
                </div>
            </div>

            <!-- 评价区域 -->
            <div class="evaluations-section">
                <Evaluations :contentId="cookbook.id" contentType="COOKBOOK" />
            </div>
        </div>
    </div>
</template>

<script>
import Evaluations from "@/components/Evaluations"

export default {
    name: "CookbookDetail",
    components: { Evaluations },
    
    data() {
        return {
            cookbook: {
                id: '',
                title: '',
                categoryName: '',
                createTime: '',
                content: '',
                // 结构化数据字段
                ingredients: [],
                steps: [],
                // 原始数据
                rawData: {}
            }
        }
    },
    
    computed: {
        // 检查是否有结构化数据
        hasStructuredData() {
            return this.ingredientsData.length > 0 || this.stepsData.length > 0;
        },
        
        // 获取材料数据
        ingredientsData() {
            // 优先使用结构化数据
            if (this.cookbook.ingredients && Array.isArray(this.cookbook.ingredients)) {
                return this.cookbook.ingredients;
            }
            
            // 如果没有结构化数据，尝试从原始内容中提取
            if (this.cookbook.content && this.cookbook.content.includes('table')) {
                return this.extractIngredientsFromHTML(this.cookbook.content);
            }
            
            return [];
        },
        
        // 获取步骤数据
        stepsData() {
            if (this.cookbook.steps && Array.isArray(this.cookbook.steps)) {
                return this.cookbook.steps;
            }
            return [];
        }
    },
    
    created() {
        this.loadCookbookDetail();
    },
    
    methods: {
        goBack() {
            this.$router.go(-1);
        },
        
        loadCookbookDetail() {
            try {
                const jsonCookbook = sessionStorage.getItem('cookbookInfo');
                if (jsonCookbook) {
                    const cookbookData = JSON.parse(jsonCookbook);
                    this.cookbook = {
                        ...cookbookData,
                        rawData: cookbookData
                    };
                    
                    // 记录数据格式，便于调试
                    console.log('食谱数据格式:', {
                        有HTML表格: cookbookData.content && cookbookData.content.includes('<table'),
                        有结构化材料数据: cookbookData.ingredients && cookbookData.ingredients.length > 0,
                        有结构化步骤数据: cookbookData.steps && cookbookData.steps.length > 0
                    });
                } else {
                    console.warn('sessionStorage中未找到cookbookInfo');
                }
            } catch (error) {
                console.error('解析食谱数据失败:', error);
            }
        },
        
        // 处理HTML表格，添加样式
        processTableHTML(htmlContent) {
            if (!htmlContent) return '';
            
            // 为表格添加CSS样式
            let processedHTML = htmlContent
                .replace(/<table>/g, '<table class="html-table">')
                .replace(/<th>/g, '<th style="background-color: #f2f2f2; font-weight: bold; padding: 12px; border: 1px solid #ddd;">')
                .replace(/<td>/g, '<td style="padding: 8px 12px; border: 1px solid #ddd;">')
                .replace(/<tr>/g, '<tr style="border-bottom: 1px solid #eee;">')
                .replace(/<table>/g, '<table style="width: 100%; border-collapse: collapse; margin: 20px 0; border: 1px solid #ddd;">');
            
            return processedHTML;
        },
        
        // 从HTML中提取材料数据
        extractIngredientsFromHTML(htmlContent) {
            try {
                // 创建临时DOM元素
                const tempDiv = document.createElement('div');
                tempDiv.innerHTML = htmlContent;
                
                const tables = tempDiv.getElementsByTagName('table');
                if (tables.length === 0) return [];
                
                const firstTable = tables[0];
                const rows = firstTable.getElementsByTagName('tr');
                const ingredients = [];
                
                // 跳过表头行
                for (let i = 1; i < rows.length; i++) {
                    const cells = rows[i].getElementsByTagName('td');
                    if (cells.length >= 3) {
                        ingredients.push({
                            category: cells[0].textContent || '',
                            ingredient: cells[1].textContent || '',
                            remark: cells[2].textContent || ''
                        });
                    }
                }
                
                return ingredients;
            } catch (error) {
                console.error('从HTML提取材料数据失败:', error);
                return [];
            }
        },
        
        formatDate(dateString) {
            if (!dateString) return '未指定日期';
            try {
                const date = new Date(dateString);
                if (isNaN(date.getTime())) {
                    return dateString;
                }
                return date.toLocaleDateString('zh-CN', {
                    year: 'numeric',
                    month: 'long',
                    day: 'numeric'
                });
            } catch (error) {
                return dateString;
            }
        }
    }
};
</script>



<style scoped lang="scss">
.back-button:hover {
    background-color: #FFF5E6;
    border-color: #C28B59;
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(194, 145, 89, 0.2);
}

.cookbook-container {
    max-width: 1200px;
    margin: 0 auto;
    background-color: #fff;
    border-radius: 12px;
    padding: 40px;
    box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
    
    .cookbook-header {
        margin-bottom: 40px;
        padding-bottom: 20px;
        border-bottom: 2px solid #E8D4B9;
        
        .cookbook-title {
            font-size: 36px;
            font-weight: bold;
            color: #5C3D2E;
            margin-bottom: 20px;
            line-height: 1.3;
        }
        
        .cookbook-meta {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            font-size: 16px;
            color: #8C6B4A;
            
            .meta-item {
                display: flex;
                align-items: center;
                gap: 8px;
                
                i {
                    font-size: 18px;
                    color: #C28B59;
                }
            }
        }
    }
    
    .cookbook-content {
        .content-section {
            margin-bottom: 40px;
            
            &.html-content {
                :deep(table.html-table) {
                    width: 100%;
                    border-collapse: collapse;
                    margin: 20px 0;
                    box-shadow: 0 2px 8px rgba(194, 145, 89, 0.1);
                    border-radius: 8px;
                    overflow: hidden;
                    background-color: white;
                    
                    th, td {
                        border: 1px solid #E8D4B9;
                        padding: 12px 16px;
                        text-align: left;
                        font-size: 14px;
                        line-height: 1.5;
                    }
                    
                    th {
                        background-color: #FFF9F0;
                        font-weight: 600;
                        color: #5C3D2E;
                        white-space: nowrap;
                    }
                    
                    td {
                        color: #8C6B4A;
                        word-break: break-word;
                        
                        // 高亮特定内容，如图片中的"鸡蛋"
                        &:contains("鸡蛋") {
                            background-color: #FFF5E6;
                            color: #A8723A;
                            font-weight: 500;
                        }
                    }
                    
                    tr:nth-child(even) {
                        background-color: #FAF6F0;
                    }
                    
                    tr:hover {
                        background-color: #FFF5E6;
                        transition: background-color 0.3s;
                    }
                }
                
                // 为整个HTML内容添加样式
                :deep(*) {
                    line-height: 1.8;
                    color: #8C6B4A;
                    
                    h1, h2, h3, h4 {
                        color: #5C3D2E;
                        margin-top: 30px;
                        margin-bottom: 15px;
                    }
                    
                    p {
                        margin-bottom: 16px;
                    }
                    
                    img {
                        max-width: 100%;
                        height: auto;
                        border-radius: 8px;
                        margin: 20px 0;
                        display: block;
                    }
                    
                    ul, ol {
                        margin: 16px 0;
                        padding-left: 24px;
                        
                        li {
                            margin-bottom: 8px;
                        }
                    }
                }
            }
            
            &.structured-content {
                .section {
                    margin-bottom: 40px;
                    
                    .section-title {
                        font-size: 24px;
                        color: #5C3D2E;
                        margin-bottom: 20px;
                        padding-bottom: 10px;
                        border-bottom: 2px solid #C28B59;
                        display: flex;
                        align-items: center;
                        gap: 10px;
                        
                        i {
                            color: #C28B59;
                        }
                    }
                    
                    .table-container {
                        overflow-x: auto;
                        margin-bottom: 20px;
                        
                        .ingredients-table {
                            min-width: 600px;
                            border-radius: 8px;
                            overflow: hidden;
                            box-shadow: 0 2px 8px rgba(194, 145, 89, 0.1);
                            
                            :deep(.el-table__header-wrapper) {
                                th {
                                    background-color: #FFF9F0;
                                    color: #5C3D2E;
                                    font-weight: 600;
                                }
                            }
                            
                            :deep(.el-table__row) {
                                td {
                                    font-size: 14px;
                                    color: #8C6B4A;
                                }
                            }
                            
                            :deep(.el-table__row:hover) {
                                td {
                                    background-color: #FFF5E6;
                                }
                            }
                        }
                    }
                    
                    &.steps-section {
                        .steps-list {
                            .step-item {
                                display: flex;
                                gap: 16px;
                                padding: 20px;
                                background-color: white;
                                border-radius: 8px;
                                margin-bottom: 16px;
                                box-shadow: 0 2px 8px rgba(194, 145, 89, 0.05);
                                border-left: 4px solid #C28B59;
                                transition: all 0.3s;
                                
                                &:hover {
                                    transform: translateY(-2px);
                                    box-shadow: 0 4px 12px rgba(194, 145, 89, 0.1);
                                }
                                
                                .step-number {
                                    display: flex;
                                    align-items: center;
                                    justify-content: center;
                                    width: 40px;
                                    height: 40px;
                                    background: linear-gradient(135deg, #C28B59, #D4A55E);
                                    color: white;
                                    border-radius: 50%;
                                    font-weight: bold;
                                    font-size: 18px;
                                    flex-shrink: 0;
                                }
                                
                                .step-content {
                                    flex: 1;
                                    
                                    .step-desc {
                                        font-size: 16px;
                                        color: #5C3D2E;
                                        margin-bottom: 8px;
                                        line-height: 1.6;
                                    }
                                    
                                    .step-time, .step-tip {
                                        font-size: 14px;
                                        color: #8C6B4A;
                                        margin-top: 4px;
                                        display: flex;
                                        align-items: center;
                                        gap: 6px;
                                    }
                                }
                            }
                        }
                    }
                }
            }
            
            &.text-content {
                .text-content-wrapper {
                    line-height: 1.8;
                    color: #8C6B4A;
                    font-size: 16px;
                    white-space: pre-wrap;
                    background-color: #FFF9F0;
                    padding: 24px;
                    border-radius: 8px;
                    border-left: 4px solid #C28B59;
                }
            }
        }
        
        .empty-content {
            text-align: center;
            padding: 60px 20px;
            color: #8C6B4A;
            
            i {
                font-size: 60px;
                margin-bottom: 20px;
                color: #E8D4B9;
            }
            
            p {
                font-size: 16px;
            }
        }
    }
    
    .evaluations-section {
        border-top: 2px solid #E8D4B9;
        padding-top: 40px;
    }
}

// 响应式设计
@media (max-width: 768px) {
    .cookbook-container {
        padding: 20px;
        margin: 0 10px;
        
        .cookbook-header {
            .cookbook-title {
                font-size: 28px;
            }
            
            .cookbook-meta {
                flex-direction: column;
                gap: 10px;
            }
        }
        
        .cookbook-content {
            .content-section {
                &.structured-content {
                    .section {
                        .section-title {
                            font-size: 20px;
                        }
                        
                        .table-container {
                            margin: 0 -20px;
                            padding: 0 20px;
                            
                            .ingredients-table {
                                min-width: 500px;
                            }
                        }
                        
                        &.steps-section {
                            .steps-list {
                                .step-item {
                                    padding: 16px;
                                    
                                    .step-number {
                                        width: 32px;
                                        height: 32px;
                                        font-size: 16px;
                                    }
                                    
                                    .step-content {
                                        .step-desc {
                                            font-size: 15px;
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
</style>