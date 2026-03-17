<template>
    <div style="background-color: #FFF9F0;padding: 20px;min-height: 100vh;">
        <el-row>
            <el-col :span="18">
                <div style="margin: 40px 0;">
                    <span style="font-size: 24px;color: #5C3D2E;font-weight: bold;">粤菜美食帖子</span>
                    <el-input size="small" style="width: 266px;float: right;" v-model="gourtmetQueryDto.title"
                        placeholder="搜索美食" clearable @clear="handleFilterClear">
                        <el-button slot="append" @click="fetchGourmetData" icon="el-icon-search" 
                                  style="background-color: #C28B59;border-color: #C28B59;color: white;"></el-button>
                    </el-input>
                </div>
                <div style="margin-block: 20px;">
                    <span :style="{
                        backgroundColor: categoryClick.id === category.id ? '#C28B59' : '#FFF5E6',
                        color: categoryClick.id === category.id ? '#FFF9F0' : '#5C3D2E',
                        borderColor: categoryClick.id === category.id ? '#A8723A' : '#E8D4B9'
                    }" @click="categorySelected(category)" class="item-category"
                        v-for="(category, index) in categories" :key="index">
                        {{ category.name }}
                    </span>
                </div>
                <div v-if="gourmetList.length === 0">
                    <el-empty description="美食做法暂时未找到"></el-empty>
                </div>
                <div v-else>
                    <div style="display: flex;justify-content: left;border-bottom: 1px solid #E8D4B9;padding: 20px 0;margin-bottom: 20px;background-color: white;border-radius: 8px;padding: 20px;box-shadow: 0 2px 8px rgba(194, 145, 89, 0.1);"
                        class="item-gourmet" v-for="(gourmet, index) in gourmetList" :key="index">
                        <div class="left">
                            <img :src="gourmet.cover" style="width: 220px;height: 150px;border-radius: 8px;object-fit: cover;">
                        </div>
                        <div class="right" style="flex: 1;margin-left: 20px;">
                            <div class="info" style="display: flex;align-items: center;margin-bottom: 12px;">
                                <img style="width: 32px;height: 32px;border-radius: 16px;margin-right: 8px;border: 2px solid #FFE9C8;" 
                                    :src="gourmet.userAvatar" alt="">
                                <span style="font-weight: 600;color: #5C3D2E;">{{ gourmet.userName }}</span>
                            </div>
                            <div style="font-size: 20px;font-weight: 700;color: #5C3D2E;margin-bottom: 12px;cursor: pointer;" 
                                class="title" @click="readGourmet(gourmet)">
                                {{ gourmet.title }}
                            </div>
                            <div style="color: #8C6B4A;font-size: 15px;line-height: 1.6;margin-bottom: 16px;max-height: 72px;overflow: hidden;text-overflow: ellipsis;display: -webkit-box;-webkit-line-clamp: 3;-webkit-box-orient: vertical;"
                                 v-if="gourmet.detail && gourmet.detail.length > 0">
                                {{ gourmet.detail }}
                            </div>
                            <div style="display: flex;align-items: center;color: #8C6B4A;font-size: 13px;margin-top: 12px;">
                                <span style="margin-right: 20px;background-color: #FFF5E6;padding: 4px 12px;border-radius: 12px;border: 1px solid #E8D4B9;">
                                    {{ formatTime(gourmet.createTime) }}
                                </span>
                                <span style="display: flex;align-items: center;margin-right: 20px;cursor: pointer;color: #A8723A;">
                                    <i class="el-icon-discount" style="margin-right: 6px;font-size: 16px;"></i>
                                    ({{ gourmet.upvoteCount || 0 }})
                                </span>
                                <span style="display: flex;align-items: center;margin-right: 20px;cursor: pointer;color: #A8723A;">
                                    <i class="el-icon-view" style="margin-right: 6px;font-size: 16px;"></i>
                                    ({{ gourmet.viewCount || 0 }})
                                </span>
                                <span style="display: flex;align-items: center;cursor: pointer;color: #A8723A;">
                                    <i class="el-icon-star-off" style="margin-right: 6px;font-size: 16px;"></i>
                                    ({{ gourmet.saveCount || 0 }})
                                </span>
                            </div>
                        </div>
                    </div>
                    <el-pagination style="margin: 20px 0;text-align: center;" @size-change="handleSizeChange"
                        @current-change="handleCurrentChange" :current-page="currentPage" :page-sizes="[10, 20]"
                        :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper" :total="totalItems"
                        :background="true"
                        :page-count="Math.ceil(totalItems / pageSize)"
                        :pager-count="5"
                        :hide-on-single-page="false">
                    </el-pagination>
                </div>
            </el-col>
            <el-col :span="6" style="padding-left: 30px;">
                <div style="padding: 20px;background: white;border-radius: 8px;position: sticky;top: 60px;border: 1px solid #E8D4B9;box-shadow: 0 2px 8px rgba(194, 145, 89, 0.1);">
                    <p style="font-size: 20px;font-weight: 700;color: #5C3D2E;margin-bottom: 20px;padding-bottom: 12px;border-bottom: 2px solid #FFE9C8;">猜你想看</p>
                    <div style="display: flex;flex-direction: column;gap: 16px;">
                        <div v-for="(gourmet, index) in topList" :key="index" 
                             style="display: flex;cursor: pointer;padding-bottom: 16px;border-bottom: 1px solid #F0E6D9;transition: all 0.3s;"
                             @click="readGourmet(gourmet)"
                             class="recommend-item">
                            <div style="margin-right: 12px;position: relative;overflow: hidden;border-radius: 6px;">
                                <img :src="gourmet.cover" 
                                     style="width: 100px;height: 80px;border-radius: 6px;object-fit: cover;transition: transform 0.3s;">
                            </div>
                            <div style="flex: 1;min-width: 0;">
                                <div style="font-size: 16px;font-weight: 700;color: #5C3D2E;margin-bottom: 8px;line-height: 1.3;white-space: nowrap;overflow: hidden;text-overflow: ellipsis;"
                                     class="title">{{ gourmet.title }}</div>
                                <div style="display: flex;align-items: center;color: #8C6B4A;font-size: 13px;margin-bottom: 6px;">
                                    <img :src="gourmet.userAvatar" 
                                         style="width: 20px;height: 20px;border-radius: 10px;margin-right: 8px;border: 2px solid #FFE9C8;">
                                    <span style="font-weight: 500;">{{ gourmet.userName }}</span>
                                </div>
                                <div style="display: flex;align-items: center;color: #A8723A;font-size: 12px;">
                                    <span style="background: #FFF5E6;padding: 4px 10px;border-radius: 12px;border: 1px solid #E8D4B9;margin-right: 8px;font-weight: 500;">
                                        {{ gourmet.categoryName }}
                                    </span>
                                    <span>{{ timeOut(gourmet.createTime) }}</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </el-col>
        </el-row>
    </div>
</template>
<script>
import { timeAgo } from "@/utils/data"
export default {
    name: "Gourmet",
    data() {
        return {
            gourtmetQueryDto: {},
            gourmetList: [
                {
                    id: 1,
                    userAvatar: 'https://randomuser.me/api/portraits/men/32.jpg',
                    userName: '张无忌',
                    title: '风味腊肠',
                    detail: '到底是谁发明的这个喝法! 谢谢啊~ 真的又简单又好喝! 以后谁还去奶茶店啊! 红茶+白糖炒一炒，炒到焦黄冒泡泡，加一点水煮开，最后倒牛奶就完成啦! (全程小火，糖炒的焦黄冒泡泡就赶紧加水! 不要炒过头，我就炒糊一次 ) 家里有小朋友的做给小朋友喝也很干净卫生~ 家里有珍珠的可以加珍珠，有啥小料就放啥! 真的超级无敌好喝! 赶快试试! !',
                    createTime: '2024-12-04 17:45:38',
                    upvoteCount: 1,
                    viewCount: 3,
                    saveCount: 1,
                    cover: 'https://images.unsplash.com/photo-1565299624946-b28f40a0ae38?ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=80'
                },
                {
                    id: 2,
                    userAvatar: 'https://randomuser.me/api/portraits/women/44.jpg',
                    userName: '赵敏',
                    title: '干香蛙',
                    detail: '奶羹，超级好吃! 这道菜的关键在于火候的控制，要先用中火煎至金黄，再用小火慢炖，让蛙肉充分吸收汤汁的精华。配上特制的香料，香气扑鼻，口感鲜嫩。',
                    createTime: '404天前',
                    upvoteCount: 5,
                    viewCount: 12,
                    saveCount: 3,
                    cover: 'https://images.unsplash.com/photo-1565299507177-b0ac66763828?ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=60'
                }
            ],
            categories: [
                { id: null, name: '全部' },
                { id: 1, name: '客家菜' },
                { id: 2, name: '潮汕菜' },
                { id: 3, name: '广府菜' }
            ],
            filterText: '',
            currentPage: 1,
            pageSize: 10,
            totalItems: 2,
            topList: [
                {
                    id: 3,
                    userAvatar: 'https://randomuser.me/api/portraits/men/32.jpg',
                    userName: '张无忌',
                    title: '风味腊肠',
                    categoryName: '客家菜',
                    createTime: '404天前',
                    cover: 'https://images.unsplash.com/photo-1565299624946-b28f40a0ae38?ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=80'
                },
                {
                    id: 4,
                    userAvatar: 'https://randomuser.me/api/portraits/women/44.jpg',
                    userName: '赵敏',
                    title: '干香蛙',
                    categoryName: '广府菜',
                    createTime: '405天前',
                    cover: 'https://images.unsplash.com/photo-1565299507177-b0ac66763828?ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=60'
                },
                {
                    id: 5,
                    userAvatar: 'https://randomuser.me/api/portraits/men/32.jpg',
                    userName: '张无忌',
                    title: '奶羹',
                    categoryName: '潮汕菜',
                    createTime: '404天前',
                    cover: 'https://images.unsplash.com/photo-1556909114-f6e7ad7d3136?ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=80'
                }
            ],
            defaultLoadItem: 5,
            categoryClick: { id: null, name: '全部' }
        }
    },
    created() {
        this.fetchCategoryData();
        this.fetchGourmetData();
        this.loadRecommend(this.defaultLoadItem);
    },
    methods: {
        timeOut(time) {
            return timeAgo(time);
        },
        
        formatTime(time) {
            if (!time) return '';
            // 如果是"X天前"的格式，直接返回
            if (time.includes('天前')) {
                return time;
            }
            // 否则尝试解析为日期
            try {
                const date = new Date(time);
                if (!isNaN(date.getTime())) {
                    return date.toLocaleDateString('zh-CN');
                }
            } catch (e) {
                // 解析失败，返回原始值
            }
            return time.split(' ')[0];
        },
        
        loadRecommend(item) {
            // 模拟API调用
            this.$axios.get(`/recommend/${item}`).then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.topList = res.data.data;
                }
            }).catch(error => {
                console.log(error);
            });
        },
        readGourmet(gourmet) {
            sessionStorage.setItem('gourmetId', gourmet.id);
            this.$router.push('/gourmetDetail');
        },
        handleFilterClear() {
            this.filterText = '';
            this.fetchGourmetData();
        },
        // 页面大小改变
        handleSizeChange(val) {
            this.pageSize = val;
            this.currentPage = 1;
            this.fetchGourmetData();
        },
        // 当前页切换
        handleCurrentChange(val) {
            this.currentPage = val;
            this.fetchGourmetData();
        },
        categorySelected(category) {
            this.categoryClick = category;
            this.gourtmetQueryDto.categoryId = category.id;
            this.fetchGourmetData();
        },
        // 查询美食类别
        fetchCategoryData() {
            // 模拟API调用
            this.$axios.post('/category/query', {}).then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.categories = res.data.data;
                    this.categories.unshift(this.categoryClick);
                }
            }).catch(error => {
                console.log(error);
            });
        },
        // 查询既公开又审核的美食做法
        fetchGourmetData() {
            const queryDto = {
                current: this.currentPage,
                size: this.pageSize,
                key: this.filterText,
                ...this.gourtmetQueryDto
            };
            this.$axios.post('/gourmet/queryList', queryDto).then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.gourmetList = res.data.data;
                    this.totalItems = data.total;
                }
            }).catch(error => {
                console.log(error);
            });
        },
    }
};
</script>
<style scoped lang="scss">
.item-category {
    display: inline-block;
    padding: 8px 20px;
    border-radius: 20px;
    margin-right: 8px;
    margin-bottom: 8px;
    font-size: 14px;
    cursor: pointer;
    transition: all 0.3s ease;
    border: 1px solid #E8D4B9;
    font-weight: 600;
    
    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(194, 145, 89, 0.2);
        background: linear-gradient(135deg, rgba(212, 165, 94, 0.1) 0%, rgba(194, 145, 89, 0.1) 100%);
    }
}

.title:hover {
    color: #C28B59;
}

:deep(.el-input__inner) {
    border: 1px solid #E8D4B9;
    background: white;
    color: #5C3D2E;
    &:focus {
        border-color: #C28B59;
        box-shadow: 0 0 0 2px rgba(194, 145, 89, 0.1);
    }
}

:deep(.el-pagination.is-background .el-pager li:not(.disabled).active) {
    background-color: #C28B59;
    border-color: #A8723A;
}

:deep(.el-pagination.is-background .el-pager li:not(.disabled):hover) {
    color: #C28B59;
}

:deep(.el-button--primary) {
    background-color: #C28B59;
    border-color: #C28B59;
}

:deep(.el-button--primary:hover) {
    background-color: #A8723A;
    border-color: #A8723A;
}

.recommend-item:hover {
    transform: translateX(4px);
    
    img {
        transform: scale(1.05);
    }
    
    .title {
        color: #C28B59;
    }
}

.item-gourmet:hover {
    box-shadow: 0 4px 20px rgba(194, 145, 89, 0.15);
    transform: translateY(-2px);
    border-color: #D4A55E;
}

/* 确保整个页面背景是暖黄色 */
body {
    background-color: #FFF9F0;
}
</style>