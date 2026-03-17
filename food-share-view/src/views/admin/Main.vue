<template>
    <div style="box-sizing: border-box;overflow-y: hidden;overflow-x: hidden;">
        <el-row>
            <el-col :span="16">
                <div>
                    <div>
                        <el-row style="padding-block: 20px;margin-inline: 30px;background-color: #fef7e0; border-radius: 8px; border: 1px solid #f5e6d3;">
                            <el-col :span="4" v-for="(staticItem, index) in staticData" :key="index">
                                <div class="static-item" style="padding: 0 10px;">
                                    <el-statistic group-separator="," :precision="0" :value="staticItem.count"
                                        :title="staticItem.name"></el-statistic>
                                </div>
                            </el-col>
                        </el-row>
                    </div>
                    <div>
                        <div style="padding: 20px 60px;box-sizing: border-box;">
                            <div style="background-color: white; border-radius: 8px; padding: 20px; border: 1px solid #f5e6d3;">
                                <LineChart height="280px" tag="内容分享趋势" @on-selected="netItemSelected" :values="shareValues"
                                    :date="shareDates" />
                            </div>
                        </div>
                        <div style="padding: 20px 60px;box-sizing: border-box;">
                            <div style="background-color: white; border-radius: 8px; padding: 20px; border: 1px solid #f5e6d3;">
                                <LineChart height="280px" tag="收录美食做法" @on-selected="gourmetItemSelected"
                                    :values="gourmetValues" :date="gourmetDates" />
                            </div>
                        </div>
                    </div>
                </div>
            </el-col>
            <el-col :span="8">
                <div class="buttom" style="background-color: white; border-radius: 8px; padding: 20px; margin-right: 20px; border: 1px solid #f5e6d3;">
                    <p style="font-size: 20px; font-weight: 700; color: #8b4513; margin-bottom: 20px; padding-bottom: 12px; border-bottom: 2px solid #f5e6d3;">最热帖子</p>
                    <div class="gourmet-item" v-for="(gourmet, index) in gourmetList" :key="index">
                        <div class="left">
                            <img :src="gourmet.cover" :alt="gourmet.title" srcset="" style="border: 1px solid #f5e6d3;">
                        </div>
                        <div class="right">
                            <div class="user">
                                <img :src="gourmet.userAvatar" alt="" srcset="" style="border: 1px solid #f5e6d3;">
                                <span style="color: #8b4513; font-weight: 500;">{{ gourmet.userName }}</span>
                            </div>
                            <div class="title" @click="readGourmet(gourmet)" style="color: #8b4513;">{{ gourmet.title }}</div>
                            <div class="info">
                                <span class="category-tag" style="color: #d2691e; background-color: #fef7e0; border: 1px solid #f5e6d3;">
                                    {{ gourmet.categoryName }}
                                </span>
                                <div class="interaction-stats">
                                    <span class="stat-item" style="color: #a0522d;">
                                        <i class="el-icon-discount" style="margin-right: 2px; font-size: 12px;"></i>
                                        {{ gourmet.upvoteCount || 0 }}
                                    </span>
                                    <span class="stat-item" style="color: #a0522d;">
                                        <i class="el-icon-view" style="margin-right: 2px; font-size: 12px;"></i>
                                        {{ gourmet.viewCount || 0 }}
                                    </span>
                                    <span class="stat-item" style="color: #a0522d;">
                                        <i class="el-icon-star-off" style="margin-right: 2px; font-size: 12px;"></i>
                                        {{ gourmet.saveCount || 0 }}
                                    </span>
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
import LineChart from "@/components/LineChart"
import { timeAgo } from "@/utils/data"
export default {
    components: { LineChart },
    data() {
        return {
            staticData: [],
            gourmetList: [],
            shareValues: [],
            shareDates: [],
            gourmetValues: [],
            gourmetDates: [],
            defaultQueryDays: 3650, 
        }
    },
    created() {
        this.fentchStaticCount();
        this.fentchgourmet();
        this.netItemSelected(this.defaultQueryDays);
        this.gourmetItemSelected(this.defaultQueryDays);
    },
    methods: {
        timeOut(time) {
            return timeAgo(time);
        },
        // 查询指定时间内的美食做法存量
        gourmetItemSelected(day) {
            this.$axios.get(`/gourmet/daysQuery/${day}`).then(res => {
                if (res.data.code === 200) {
                    this.gourmetValues = res.data.data.map(money => money.count);
                    this.gourmetDates = res.data.data.map(money => money.name);
                }
            });
        },
        // 查询指定时间内的内容分享量
        netItemSelected(day) {
            this.$axios.post(`/contentNet/daysQuery`, { day }).then(res => {
                if (res.data.code === 200) {
                    this.shareValues = res.data.data.map(money => money.count);
                    this.shareDates = res.data.data.map(money => money.name);
                }
            });
        },
        readGourmet(gourmet) {
            sessionStorage.setItem('gourmetId', gourmet.id);
            this.$router.push('/seeGourmetDetail');
        },
        // 查询阅读量最高的美食做法数据
        fentchgourmet() {
            const gourmetQueryDto = {
                current: 1,
                size: 5
            }
            this.$axios.post('/gourmet/queryByView', gourmetQueryDto).then(res => {
                if (res.data.code === 200) {
                    this.gourmetList = res.data.data;
                }
            });
        },
        // 查询静态数据
        fentchStaticCount() {
            this.$axios.get('/views/staticControls').then(res => {
                if (res.data.code === 200) {
                    this.staticData = res.data.data;
                }
            });
        },
    },
};
</script>
<style scoped lang="scss">
/* 统计组件样式 */
:deep(.el-statistic) {
    .el-statistic__head {
        color: #a0522d;
        font-weight: 600;
    }
    
    .el-statistic__content {
        color: #d2691e;
        font-weight: 700;
    }
}

.buttom {
    .gourmet-item {
        display: flex;
        justify-content: flex-start;
        padding: 10px 0;
        cursor: pointer;
        border-bottom: 1px solid #f5e6d3;
        
        &:last-child {
            border-bottom: none;
        }
        
        &:hover {
            background-color: #fef7e0;
            border-radius: 6px;
        }

        .left {
            flex-shrink: 0;
            padding: 5px;
            box-sizing: border-box;

            img {
                width: 138px;
                height: 80px;
                border-radius: 5px;
                object-fit: cover;
            }
        }

        .right {
            flex: 1;
            min-width: 0; /* 防止内容溢出 */
            padding: 5px 5px 5px 10px;
            box-sizing: border-box;
            display: flex;
            flex-direction: column;

            .user {
                display: flex;
                justify-content: flex-start;
                align-items: center;
                margin-bottom: 6px;
                flex-shrink: 0;

                img {
                    width: 20px;
                    height: 20px;
                    border-radius: 10px;
                    flex-shrink: 0;
                }

                span {
                    margin-left: 6px;
                    font-size: 13px;
                    white-space: nowrap;
                    overflow: hidden;
                    text-overflow: ellipsis;
                }
            }

            .title {
                font-size: 15px;
                font-weight: 700;
                margin-bottom: 6px;
                line-height: 1.4;
                transition: color 0.3s ease;
                /* 多行文本截断 - 修复兼容性警告 */
                display: -webkit-box;
                -webkit-line-clamp: 2;
                -webkit-box-orient: vertical;
                line-clamp: 2; /* 添加标准属性 */
                overflow: hidden;
                text-overflow: ellipsis;
                max-height: 42px; /* 2行高度 */
            }

            .title:hover {
                color: #d2691e;
            }

            .info {
                margin-top: auto; /* 将信息区域推到容器底部 */
                display: flex;
                flex-wrap: wrap;
                align-items: center;
                gap: 8px;
                
                .category-tag {
                    display: inline-block;
                    padding: 3px 8px;
                    border-radius: 12px;
                    font-size: 11px;
                    font-weight: 600;
                    white-space: nowrap;
                    flex-shrink: 0;
                }
                
                .interaction-stats {
                    display: flex;
                    align-items: center;
                    flex-wrap: nowrap;
                    gap: 12px;
                    margin-top: 2px;
                    
                    .stat-item {
                        display: flex;
                        align-items: center;
                        font-size: 11px;
                        font-weight: 500;
                        white-space: nowrap;
                        flex-shrink: 0;
                        
                        i {
                            font-size: 12px;
                            color: #c28b59;
                        }
                    }
                }
            }
        }
    }
}
</style>