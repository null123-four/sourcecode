<template>
    <div>
        <el-row style="background-color: #fffaf0;padding: 20px;">
            <el-col :span="24">
                <div class="info">
                    <div>
                        <img :src="info.userAvatar" alt="">
                    </div>
                    <div>
                        <div class="title">{{ info.userName }}</div>
                        <div class="point-info">
                            <span>
                                <span class="count">{{ info.viewCount }}</span>阅读量
                            </span>
                            <span>
                                <span class="count">
                                    {{ info.upvoteCount }}
                                </span>
                                获赞量
                            </span>
                            <span>
                                <span class="count">{{ info.saveCount }}</span>
                                收藏量
                            </span>
                        </div>
                    </div>
                </div>
                <div style="margin-inline: 20px;">
                    <el-tabs v-model="activeName" @tab-click="handleClick" class="warm-yellow-tabs">
                        <el-tab-pane label="美食帖子" name="first">
                            <div style="display: flex;justify-content: left;" class="item-gourmet"
                                v-for="(gourmet, index) in gourmetList" :key="index">
                                <div class="left">
                                    <img :src="gourmet.cover">
                                </div>
                                <div class="right">
                                    <div class="info">
                                        <img style="width: 20px;height: 20px;border-radius: 10px;"
                                            :src="gourmet.userAvatar" alt="">
                                        <span>{{ gourmet.userName }}</span>
                                    </div>
                                    <div class="title" @click="readGourmet(gourmet)">
                                        {{ gourmet.title }}
                                    </div>
                                    <div class="detail">
                                        {{ gourmet.detail }}
                                    </div>
                                    <div class="detail">
                                        <span>
                                            {{ gourmet.createTime }}
                                        </span>
                                        <span>
                                            <i class="el-icon-discount" style="margin-right: 4px;"></i>({{
                                                gourmet.upvoteCount }})
                                        </span>
                                        <span>
                                            <i class="el-icon-view" style="margin-right: 4px;"></i>({{ gourmet.viewCount
                                            }})
                                        </span>
                                        <span>
                                            <i class="el-icon-star-off" style="margin-right: 4px;"></i>({{
                                                gourmet.saveCount
                                            }})
                                        </span>
                                        <span class="action-button" @click="share(gourmet)">
                                            分享
                                        </span>
                                        <span class="action-button" @click="edit(gourmet)">
                                            修改
                                        </span>
                                        <span class="action-button delete-button" @click="del(gourmet)">
                                            删除
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </el-tab-pane>
                        <el-tab-pane label="食谱" name="second">
                            <MyCookbook />
                        </el-tab-pane>
                        <el-tab-pane label="内容分享" name="third">
                            <MyContentNetManage />
                        </el-tab-pane>
                    </el-tabs>
                </div>
            </el-col>
            <el-dialog :visible.sync="dialogShareOperaion" width="20%" custom-class="warm-yellow-dialog">
                <div style="padding:16px 20px;">
                    <p class="dialog-title" v-if="url === ''">内容分享</p>
                    <div v-if="url !== ''">
                        <el-result icon="success" title="分享成功" subTitle="">
                            <template slot="extra">
                                <span class="share-url">
                                    {{ url }}
                                </span>
                            </template>
                        </el-result>
                    </div>
                    <div v-else>
                        <div>
                            <p class="dialog-subtitle">*有效期</p>
                            <el-radio-group size="mini" v-model="contentNet.validDay" class="warm-yellow-radio">
                                <el-radio-button label="3天"></el-radio-button>
                                <el-radio-button label="7天"></el-radio-button>
                                <el-radio-button label="30天"></el-radio-button>
                                <el-radio-button label="长期有效"></el-radio-button>
                            </el-radio-group>
                        </div>
                        <div>
                            <p class="dialog-subtitle">*是否启用密码认证</p>
                            <el-switch v-model="contentNet.passwordAuth" active-color="#d2691e"
                                inactive-color="#c9a880">
                            </el-switch>
                        </div>
                        <div v-if="contentNet.passwordAuth">
                            <p class="dialog-subtitle">*访问密码</p>
                            <input class="dialog-input" type="password" v-model="contentNet.accessPassword"
                                placeholder="密码" />
                        </div>
                    </div>

                </div>
                <span slot="footer" class="dialog-footer" style="margin-top: 10px;">
                    <span class="channel-button" @click="cannel()">
                        取消操作
                    </span>
                    <div v-if="url === ''">
                        <span class="edit-button" @click="addOperation()">
                            确定分享
                        </span>
                    </div>
                </span>
            </el-dialog>
        </el-row>
        <el-row  style="background-color: #fffaf0;padding: 20px;">
            <div style="padding: 30px;margin-top: 30px;">
                <Hot />
            </div>
        </el-row>
    </div>

</template>
<script>
import MyContentNetManage from "@/views/user/MyContentNetManage"
import MyCookbook from "@/views/user/MyCookbook"
import Hot from "@/views/user/Hot"
export default {
    components: { MyContentNetManage, MyCookbook, Hot },
    name: "Service",
    data() {
        return {
            contentNet: { passwordAuth: false },
            gourmet: {},
            info: {},
            activeName: 'first',
            gourmetList: [],
            dialogShareOperaion: false,
            url: '', // 返回的分享链接
        }
    },
    created() {
        this.fetchMyslefGourmet();
        this.fetchCenter();
    },
    methods: {
        addOperation() {
            const dayList = [this.contentNet.validDay];
            const validDayList = dayList.map(text => {
                const match = text.match(/\d+/g); // 使用正则表达式匹配数字
                return match ? match : []; // 如果找到数字，返回数组，否则返回空数组
            });
            if (this.contentNet.passwordAuth) {
                this.contentNet.accessPassword = this.$md5(this.contentNet.accessPassword);
            }
            const saveEntity = {
                gourmetId: this.gourmet.id,
                validDay: validDayList[0].length === 0 ? -1 : validDayList[0][0],
                passwordAuth: this.contentNet.passwordAuth,
                accessPassword: this.contentNet.accessPassword
            }
            this.$axios.post('/contentNet/save', saveEntity).then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.$notify({
                        duration: 1000,
                        title: '分享操作',
                        message: '分享成功',
                        type: 'success'
                    });
                    this.url = data.msg;
                }
            }).catch(error => {
                console.log("查询美食做法数据异常", error);
            });
        },
        cannel() {
            this.dialogShareOperaion = false;
            this.url = '';
            this.contentNet = {};
        },
        shareOperation() {

        },
        // 跳转美食帖子修改信息页面
        edit(gourmet) {
            console.log("美食：",gourmet);
            
            sessionStorage.setItem('gourmetId', gourmet.id);
            this.$router.push('/editGourmet');
        },
        share(gourmet) {
            this.gourmet = gourmet;
            this.dialogShareOperaion = true;
        },
        async del(gourmet) {
            const confirmed = await this.$swalConfirm({
                title: '删除美食做法数据',
                text: `删除后不可恢复，是否继续？`,
                icon: 'warning',
            });
            if (confirmed) {
                try {
                    let ids = [gourmet.id];
                    const response = await this.$axios.post(`/gourmet/batchDelete`, ids);
                    if (response.data.code === 200) {
                        this.$notify({
                            duration: 1000,
                            title: '信息删除',
                            message: '删除成功',
                            type: 'success'
                        });
                        this.fetchMyslefGourmet();
                        return;
                    }
                } catch (error) {
                    this.$notify.error({
                        duration: 1000,
                        title: '删除错误',
                        message: error
                    });
                    console.error(`美食做法信息删除异常：`, error);
                }
            }
        },
        handleClick(tab, event) {
            console.log(tab, event);
        },
        readGourmet(gourmet) {
            sessionStorage.setItem('gourmetId', gourmet.id);
            this.$router.push('/gourmetDetail');
        },
        fetchMyslefGourmet() {
            this.$axios.get('/gourmet/queryUser').then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.gourmetList = data.data;
                }
            }).catch(error => {
                console.log("查询美食做法数据异常", error);

            });
        },
        fetchCenter() {
            this.$axios.get('/user/center').then(res => {
                const { data } = res;
                if (data.code === 200) {
                    this.info = data.data;
                }
            }).catch(error => {
                console.log("查询用户中心数据异常", error);

            });
        },
    }
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

/* 标签页样式 */
:deep(.warm-yellow-tabs) {
    background-color: #fffaf0;
    
    .el-tabs__header {
        border-bottom: 1px solid #f5e6d3;
        background-color: #fffaf0;
    }
    
    .el-tabs__nav-wrap {
        background-color: #fffaf0;
    }
    
    .el-tabs__item {
        color: #8b4513;
        background-color: #fef7e0;
        
        &:hover {
            color: #d2691e;
            background-color: #f5e6d3;
        }
        
        &.is-active {
            color: #d2691e;
            background-color: #fffaf0;
            border-bottom: 2px solid #f0c674;
        }
    }
    
    .el-tabs__active-bar {
        background-color: #f0c674;
    }
    
    .el-tabs__nav-wrap::after {
        background-color: #f5e6d3;
    }
    
    .el-tabs__content {
        background-color: #fffaf0;
        padding: 10px;
    }
}

/* 单选按钮组样式 */
:deep(.warm-yellow-radio) {
    .el-radio-button {
        .el-radio-button__inner {
            background-color: #fffaf0;
            border-color: #f5e6d3;
            color: #8b4513;
            
            &:hover {
                color: #d2691e;
            }
        }
        
        &.is-active {
            .el-radio-button__inner {
                background-color: #f0c674;
                border-color: #e8b85c;
                color: #8b4513;
                box-shadow: -1px 0 0 0 #e8b85c;
            }
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
}

/* 结果组件样式 */
:deep(.el-result) {
    .el-result__icon svg {
        color: #f0c674;
    }
    
    .el-result__title {
        color: #8b4513;
    }
}

.share-url {
    color: #d2691e;
    font-size: 12px;
    word-break: break-all;
    padding: 8px;
    background-color: #fef7e0;
    border-radius: 4px;
    display: inline-block;
    border: 1px solid #f5e6d3;
}

.item-gourmet:hover {
    background-color: #fef7e0;
    border-radius: 5px;
    transition: all 0.3s ease;
}

.item-gourmet {
    display: flex;
    justify-content: left;
    gap: 10px;
    padding: 10px;
    border-radius: 5px;
    cursor: pointer;
    border-bottom: 1px solid #f5e6d3;

    .left {
        img {
            width: 190px;
            border-radius: 5px;
            border: 1px solid #f5e6d3;
        }
    }

    .right {
        width: 100%;
        
        .info {
            padding: 4px 6px;
            display: flex;
            justify-content: left;
            align-items: center;
            gap: 5px;
            font-size: 14px;
            color: #a0522d;
            
            img {
                border: 1px solid #f5e6d3;
            }
        }

        .title {
            font-size: 20px;
            font-weight: 800;
            color: #8b4513;
            margin: 8px 0;
            transition: all 0.3s ease;
        }

        .title:hover {
            text-decoration: underline;
            text-decoration-style: dashed;
            color: #d2691e;
        }

        .detail {
            font-size: 12px;
            padding: 4px 6px;
            color: #a0522d;
            display: flex;
            justify-content: left;
            gap: 10px;
            flex-wrap: wrap;
            
            span {
                display: flex;
                align-items: center;
            }
        }
    }
}

.info {
    display: flex;
    justify-content: left;
    padding: 20px;
    gap: 20px;
    background-color: #fef7e0;
    border-radius: 8px;
    border: 1px solid #f5e6d3;
    margin-bottom: 20px;

    img {
        width: 100px;
        height: 100px;
        border-radius: 50px;
        border: 3px solid #f0c674;
    }

    .title {
        font-size: 26px;
        font-weight: bold;
        color: #8b4513;
        margin-bottom: 20px;
        margin-top: 5px;
    }

    .point-info {
        display: flex;
        justify-content: left;
        gap: 20px;
        
        span {
            display: flex;
            flex-direction: column;
            align-items: center;
            color: #a0522d;
        }

        .count {
            font-size: 24px;
            display: inline-block;
            margin-right: 4px;
            font-weight: bold;
            color: #d2691e;
        }
    }
}

/* 操作按钮样式 */
.action-button {
    color: #d2691e;
    cursor: pointer;
    transition: all 0.3s ease;
    padding: 2px 6px;
    border-radius: 3px;
    
    &:hover {
        background-color: #f0c674;
        color: #8b4513;
        text-decoration: underline;
    }
}

.delete-button {
    color: #d9534f;
    
    &:hover {
        background-color: #f5c6cb;
        color: #d43f3a;
    }
}

/* 弹窗输入框样式 */
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

.dialog-title {
    color: #8b4513;
    font-size: 18px;
    font-weight: bold;
    margin-bottom: 15px;
    text-align: center;
}

.dialog-subtitle {
    display: block;
    color: #a0522d;
    font-size: 14px;
    font-weight: 600;
    margin-bottom: 8px;
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
</style>