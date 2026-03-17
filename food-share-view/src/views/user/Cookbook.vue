<template>
    <div style="background-color: #FFF9F0;padding: 20px;min-height: 100vh;">
        <div style="margin: 40px 0;">
            <span style="font-size: 24px;color: #5C3D2E;font-weight: bold;">粤菜美食食谱</span>
            <el-input size="small" style="width: 366px;float: right;" v-model="cookbookQueryDto.title" placeholder="搜索食谱"
                clearable @clear="handleFilterClear" class="search-input">
                <el-button slot="append" @click="fetchCookbookData" icon="el-icon-search" 
                          style="background-color: #C28B59;border-color: #C28B59;color: white;"></el-button>
            </el-input>
        </div>
        <div style="margin-block: 20px;">
            <span :style="{
                backgroundColor: categoryClick.id === category.id ? '#C28B59' : '#FFF5E6',
                color: categoryClick.id === category.id ? '#FFF9F0' : '#5C3D2E',
                borderColor: categoryClick.id === category.id ? '#A8723A' : '#E8D4B9',
                boxShadow: categoryClick.id === category.id ? '0 4px 12px rgba(194, 145, 89, 0.3)' : '0 2px 8px rgba(232, 212, 185, 0.1)'
            }" @click="categorySelected(category)" class="item-category"
                v-for="(category, index) in categories" :key="index">
                {{ category.name }}
            </span>
        </div>
        
        <div v-if="cookbookList.length === 0">
            <el-empty description="食谱暂时未找到"></el-empty>
        </div>
        <div v-else>
            <!-- 食谱列表 - 全宽布局 -->
            <div v-for="(cookbook, index) in cookbookList" :key="index" 
                 style="display: flex;justify-content: left;border-bottom: 1px solid #E8D4B9;padding: 20px 0;margin-bottom: 20px;background-color: white;border-radius: 8px;padding: 20px;box-shadow: 0 2px 8px rgba(194, 145, 89, 0.1);"
                 class="cookbook-item">
                <div style="margin-right: 20px;">
                    <img :src="cookbook.cover" style="width: 220px;height: 150px;border-radius: 8px;object-fit: cover;" alt="">
                </div>
                <div style="flex: 1;">
                    <div style="display: flex;align-items: center;margin-bottom: 12px;">
                        <img style="width: 32px;height: 32px;border-radius: 16px;margin-right: 8px;border: 2px solid #FFE9C8;" 
                            :src="cookbook.userAvatar" alt="">
                        <span style="font-weight: 600;color: #5C3D2E;">{{ cookbook.userName }}</span>
                    </div>
                    <div @click="readDetail(cookbook)" style="cursor: pointer;font-size: 20px;font-weight: 700;color: #5C3D2E;margin-bottom: 12px;">
                        {{ cookbook.title }}
                    </div>
                    <div style="color: #8C6B4A;font-size: 15px;line-height: 1.6;margin-bottom: 16px;overflow: hidden;text-overflow: ellipsis;display: -webkit-box; -webkit-line-clamp: 3; -webkit-box-orient: vertical; line-clamp: 3;"
                         v-if="cookbook.detail && cookbook.detail.length > 0">
                        {{ cookbook.detail }}
                    </div>
                    <div style="display: flex;align-items: center;color: #8C6B4A;font-size: 13px;margin-top: 12px;">
                        <span style="margin-right: 20px;background-color: #FFF5E6;padding: 4px 12px;border-radius: 12px;border: 1px solid #E8D4B9;">
                            {{ formatTime(cookbook.createTime) }}
                        </span>
                       
                    </div>
                </div>
            </div>
            <el-pagination style="margin: 20px 0;text-align: center;" @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="currentPage" :page-sizes="[10, 20]" :page-size="pageSize"
                layout="total, sizes, prev, pager, next, jumper" :total="totalItems"
                :background="true"
                :page-count="Math.ceil(totalItems / pageSize)"
                :pager-count="5"
                :hide-on-single-page="false"
                class="custom-pagination">
            </el-pagination>
        </div>
    </div>
</template>

<script>
import { timeAgo } from "@/utils/data"
export default {
    name: "Cookbook",
    data() {
        return {
            cookbookQueryDto: {},
            cookbookList: [
                {
                    id: 1,
                    userAvatar: 'https://randomuser.me/api/portraits/men/32.jpg',
                    userName: '张无忌',
                    title: '风味腊肠',
                    detail: '到底是谁发明的这个喝法! 谢谢啊~ 真的又简单又好喝! 以后谁还去奶茶店啊! 红茶+白糖炒一炒，炒到焦黄冒泡泡，加一点水煮开，最后倒牛奶就完成啦! (全程小火，糖炒的焦黄冒泡泡就赶紧加水! 不要炒过头，我就炒糊一次 ) 家里有小朋友的做给小朋友喝也很干净卫生~ 家里有珍珠的可以加珍珠，有啥小料就放啥! 真的超级无敌好喝! 赶快试试! ! 到底是谁发明的这个喝法! 谢谢啊~',
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
                    detail: '这道菜的关键在于火候的控制，要先用中火煎至金黄，再用小火慢炖，让蛙肉充分吸收汤汁的精华。配上特制的香料，香气扑鼻，口感鲜嫩。',
                    createTime: '405天前',
                    upvoteCount: 5,
                    viewCount: 12,
                    saveCount: 3,
                    cover: 'https://images.unsplash.com/photo-1565299507177-b0ac66763828?ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=60'
                },
                {
                    id: 3,
                    userAvatar: 'https://randomuser.me/api/portraits/men/22.jpg',
                    userName: '松鼠丝',
                    title: '白炒河粉',
                    detail: '白炒河粉是广东传统小吃，关键在于炒制时的火候和调味。河粉要提前浸泡，炒的时候快速翻炒，保持河粉的弹性和口感。',
                    createTime: '405天前',
                    upvoteCount: 3,
                    viewCount: 8,
                    saveCount: 2,
                    cover: 'https://images.unsplash.com/photo-1556909114-f6e7ad7d3136?ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=80'
                },
                {
                    id: 4,
                    userAvatar: 'https://randomuser.me/api/portraits/women/33.jpg',
                    userName: '青木瓜',
                    title: '白炒牛栏山',
                    detail: '这道菜的关键在于火候的控制，要先用中火煎至金黄，再用小火慢炖，让肉充分吸收汤汁的精华。配上特制的香料，香气扑鼻，口感鲜嫩。',
                    createTime: '404天前',
                    upvoteCount: 2,
                    viewCount: 5,
                    saveCount: 1,
                    cover: 'https://images.unsplash.com/photo-1565299624946-b28f40a0ae38?ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=80'
                },
                {
                    id: 5,
                    userAvatar: 'https://randomuser.me/api/portraits/men/45.jpg',
                    userName: '张三丰',
                    title: '红烧狮子头',
                    detail: '红烧狮子头是传统名菜，肉丸鲜嫩多汁，汤汁浓郁。制作时要注意肉馅的搅拌和调味，慢火炖煮让味道充分渗透。',
                    createTime: '403天前',
                    upvoteCount: 7,
                    viewCount: 15,
                    saveCount: 4,
                    cover: 'https://images.unsplash.com/photo-1565299624946-b28f40a0ae38?ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=80'
                },
                {
                    id: 6,
                    userAvatar: 'https://randomuser.me/api/portraits/women/22.jpg',
                    userName: '小龙女',
                    title: '清蒸鲈鱼',
                    detail: '清蒸鲈鱼讲究原汁原味，鱼肉鲜嫩。蒸制时间要恰到好处，配上葱姜丝和蒸鱼豉油，简单又美味。',
                    createTime: '402天前',
                    upvoteCount: 4,
                    viewCount: 9,
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
            totalItems: 6,
            categoryClick: { id: null, name: '全部' }
        }
    },
    created() {
        this.fetchCategoryData();
        this.fetchCookbookData();
    },
    methods: {
        timeOut(time) {
            return timeAgo(time);
        },
        
        formatTime(time) {
            if (!time) return '';
            if (time.includes('天前')) {
                return time;
            }
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
        
        readDetail(cookbook) {
            sessionStorage.setItem('cookbookInfo', JSON.stringify(cookbook));
            this.$router.push('/cookbookDetail');
        },
        handleFilterClear() {
            this.filterText = '';
            this.fetchCookbookData();
        },
        // 页面大小改变
        handleSizeChange(val) {
            this.pageSize = val;
            this.currentPage = 1;
            this.fetchCookbookData();
        },
        // 当前页切换
        handleCurrentChange(val) {
            this.currentPage = val;
            this.fetchCookbookData();
        },
        categorySelected(category) {
            this.categoryClick = category;
            this.cookbookQueryDto.categoryId = category.id;
            this.fetchCookbookData();
        },
        // 查询食谱类别
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
        // 查询既公开又审核的食谱做法
        fetchCookbookData() {
            const queryDto = {
                current: this.currentPage,
                size: this.pageSize,
                key: this.filterText,
                ...this.cookbookQueryDto
            };
            this.$axios.post('/cookbook/queryPublish', queryDto).then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.cookbookList = res.data.data;
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
.search-input {
    :deep(.el-input__inner) {
        border: 1px solid #E8D4B9;
        background: white;
        color: #5C3D2E;
        border-radius: 8px 0 0 8px;
        &:focus {
            border-color: #C28B59;
            box-shadow: 0 0 0 2px rgba(194, 145, 89, 0.1);
        }
    }
    :deep(.el-input-group__append) {
        border-radius: 0 8px 8px 0;
        border: 1px solid #C28B59;
        border-left: none;
        background: linear-gradient(135deg, #D4A55E 0%, #C28B59 100%);
        color: #FFF9F0;
        &:hover {
            background: linear-gradient(135deg, #C28B59 0%, #A8723A 100%);
        }
    }
}

.custom-pagination {
    :deep(.el-pagination__total) {
        color: #8C6B4A;
    }
    :deep(.el-pagination__jump) {
        color: #8C6B4A;
    }
    :deep(.el-pager) {
        .number {
            color: #8C6B4A;
            border: 1px solid #E8D4B9;
            margin: 0 2px;
            border-radius: 6px;
            &:hover {
                color: #C28B59;
            }
            &.active {
                background: linear-gradient(135deg, #D4A55E 0%, #C28B59 100%);
                border-color: transparent;
                color: #FFF9F0;
            }
        }
    }
    :deep(.btn-prev, .btn-next) {
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
    :deep(.el-pagination__sizes) {
        .el-input {
            .el-input__inner {
                border: 1px solid #E8D4B9;
                border-radius: 6px;
                color: #5C3D2E;
                &:hover {
                    border-color: #C28B59;
                }
            }
        }
    }
    :deep(.el-pagination__editor) {
        .el-input__inner {
            border: 1px solid #E8D4B9;
            border-radius: 6px;
            color: #5C3D2E;
            &:focus {
                border-color: #C28B59;
            }
        }
    }
}

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

.cookbook-item:hover {
    box-shadow: 0 4px 20px rgba(194, 145, 89, 0.15);
    transform: translateY(-2px);
    border-color: #D4A55E;
    transition: all 0.3s ease;
}

/* 确保整个页面背景是暖黄色 */
:root {
    background-color: #FFF9F0;
}
</style>