<template>
    <div style="background-color: #FFF9F0;padding: 20px;min-height: 100vh;">
        <div v-if="collectionList.length === 0">
            <el-empty description="暂无收藏" :image-size="150"></el-empty>
        </div>
        <div v-else>
            <h3 style="font-size: 24px;color: #5C3D2E;font-weight: bold;margin-bottom: 20px;padding-bottom: 12px;border-bottom: 2px solid #E8D4B9;">
                我的收藏({{ collectionList.length }})
            </h3>
            
            <!-- 收藏列表 -->
            <div style="display: flex;flex-direction: column;gap: 20px;">
                <div v-for="(gourmet, index) in collectionList" :key="index" 
                     style="display: flex;background-color: white;border-radius: 8px;padding: 20px;box-shadow: 0 2px 8px rgba(194, 145, 89, 0.1);border: 1px solid #E8D4B9;"
                     class="item-gourmet">
                    <div class="left" style="margin-right: 20px;position: relative;overflow: hidden;border-radius: 8px;">
                        <img :src="gourmet.cover" 
                             style="width: 220px;height: 150px;border-radius: 8px;object-fit: cover;transition: transform 0.3s;">
                    </div>
                    <div class="right" style="flex: 1;display: flex;flex-direction: column;justify-content: space-between;">
                        <div>
                            <div style="display: flex;align-items: center;margin-bottom: 12px;">
                                <img style="width: 32px;height: 32px;border-radius: 16px;margin-right: 8px;border: 2px solid #FFE9C8;" 
                                    :src="gourmet.userAvatar" alt="">
                                <span style="font-weight: 600;color: #5C3D2E;font-size: 15px;">{{ gourmet.userName }}</span>
                            </div>
                            <div style="font-size: 20px;font-weight: 700;color: #5C3D2E;margin-bottom: 12px;cursor: pointer;line-height: 1.4;" 
                                class="title" @click="readGourmet(gourmet)">
                                {{ gourmet.title }}
                            </div>
                            <div style="color: #8C6B4A;font-size: 15px;line-height: 1.6;margin-bottom: 16px;overflow: hidden;text-overflow: ellipsis;display: -webkit-box;-webkit-line-clamp: 3;-webkit-box-orient: vertical;line-clamp: 3;"
                                v-if="gourmet.detail && gourmet.detail.length > 0">
                                {{ gourmet.detail }}
                            </div>
                        </div>
                        <div style="display: flex;align-items: center;color: #8C6B4A;font-size: 13px;margin-top: 12px;padding-top: 12px;border-top: 1px solid #F0E6D9;">
                            <span style="margin-right: 20px;background-color: #FFF5E6;padding: 4px 12px;border-radius: 12px;border: 1px solid #E8D4B9;color: #5C3D2E;font-weight: 500;">
                                {{ formatTime(gourmet.createTime) }}
                            </span>
                            <span style="display: flex;align-items: center;margin-right: 20px;cursor: pointer;color: #A8723A;font-weight: 500;">
                                <i class="el-icon-discount" style="margin-right: 6px;font-size: 16px;color: #C28B59;"></i>
                                {{ gourmet.upvoteCount || 0 }}
                            </span>
                            <span style="display: flex;align-items: center;margin-right: 20px;cursor: pointer;color: #A8723A;font-weight: 500;">
                                <i class="el-icon-view" style="margin-right: 6px;font-size: 16px;color: #C28B59;"></i>
                                {{ gourmet.viewCount || 0 }}
                            </span>
                            <span style="display: flex;align-items: center;cursor: pointer;color: #A8723A;font-weight: 500;">
                                <i class="el-icon-star-off" style="margin-right: 6px;font-size: 16px;color: #C28B59;"></i>
                                {{ gourmet.saveCount || 0 }}
                            </span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "Save",
    data() {
        return {
            collectionList: [],
            queryParams: {
                current: 1,
                size: 20
            },
            total: 0
        }
    },
    created() {
        this.fetchCollectionData();
    },
    methods: {
        readGourmet(gourmet) {
            sessionStorage.setItem('gourmetId', gourmet.id);
            this.$router.push('/gourmetDetail');
        },
        
        formatTime(time) {
            if (!time) return '';
            
            // 如果已经是相对时间（如"405天前"），直接返回
            if (time.includes('天前') || time.includes('小时前') || time.includes('分钟前')) {
                return time;
            }
            
            try {
                const date = new Date(time);
                if (!isNaN(date.getTime())) {
                    const now = new Date();
                    const diff = now - date;
                    const days = Math.floor(diff / (1000 * 60 * 60 * 24));
                    
                    if (days === 0) {
                        return '今天';
                    } else if (days === 1) {
                        return '昨天';
                    } else if (days < 7) {
                        return `${days}天前`;
                    } else if (days < 30) {
                        return `${Math.floor(days / 7)}周前`;
                    } else if (days < 365) {
                        return `${Math.floor(days / 30)}个月前`;
                    } else {
                        return `${Math.floor(days / 365)}年前`;
                    }
                }
            } catch (e) {
                console.error("时间格式化错误:", e);
            }
            
            return time;
        },
        
        fetchCollectionData() {
            this.$axios.post('/interaction/queryCollectionList', this.queryParams).then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.collectionList = data.data;
                    this.total = data.total || 0;
                } else {
                    this.$message.error(data.msg || '查询收藏失败');
                }
            }).catch(error => {
                console.log("查询收藏列表异常：", error);
                this.$message.error('查询失败，请稍后重试');
            });
        },
        
        // 分页方法
        handleSizeChange(val) {
            this.queryParams.size = val;
            this.queryParams.current = 1;
            this.fetchCollectionData();
        },
        
        handleCurrentChange(val) {
            this.queryParams.current = val;
            this.fetchCollectionData();
        }
    }
};
</script>

<style scoped lang="scss">
.item-gourmet {
    transition: all 0.3s ease;
    
    &:hover {
        box-shadow: 0 4px 20px rgba(194, 145, 89, 0.15);
        transform: translateY(-2px);
        border-color: #D4A55E;
    }
    
    &:hover .left img {
        transform: scale(1.05);
    }
}

.title:hover {
    color: #C28B59;
    text-decoration: underline;
    text-decoration-style: dotted;
}

/* 确保整个页面背景是暖黄色 */
body {
    background-color: #FFF9F0 !important;
}
</style>