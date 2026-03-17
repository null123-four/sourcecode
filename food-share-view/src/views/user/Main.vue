<template>
    <div class="yuecai-container">
        <!-- 顶部装饰性元素 -->
        <div class="decorative-top"></div>
        
        <div class="top">
            <!-- 添加粤菜主题背景图案 -->
            <div class="top-bg-pattern"></div>
            
            <div class="top-left">
                <Logo sysName="粤菜美食分享社区" />
                <!-- 添加粤菜小图标装饰 -->
                <div class="logo-decoration">
                    <span class="decoration-icon">🥢</span>
                    <span class="decoration-icon">🍲</span>
                </div>
            </div>
            <div class="top-right">
                <ul class="nav-menu">
                    <li @click="route('/ai-chat')" class="nav-item">
                        <span class="nav-icon">🤖</span>
                        <span class="nav-text">智能问答</span>
                    </li>
                    <li @click="route('/gourmet')" class="nav-item">
                        <span class="nav-icon">📋</span>
                        <span class="nav-text">帖子</span>
                    </li>
                    <li @click="route('/cookbook')" class="nav-item">
                        <span class="nav-icon">📕</span>
                        <span class="nav-text">食谱</span>
                    </li>
                    <li @click="route('/Save')" class="nav-item">
                        <span class="nav-icon">⭐</span>
                        <span class="nav-text">我的收藏夹</span>
                    </li>
                    <li class="nav-item dropdown-item">
                        <el-dropdown type="success" size="mini" :hide-on-click="false" class="yuecai-dropdown">
                            <span class="el-dropdown-link yuecai-dropdown-trigger">
                                <span class="nav-icon">🥗</span>
                                <span class="nav-text">我的饮食</span>
                                <i class="el-icon-arrow-down el-icon--right"></i>
                            </span>
                            <el-dropdown-menu slot="dropdown" class="yuecai-dropdown-menu">
                                <el-dropdown-item @click.native="route('/dietHistory')" class="yuecai-dropdown-item">
                                    <span class="dropdown-icon">✍️</span>记录饮食
                                </el-dropdown-item>
                                <el-dropdown-item @click.native="route('/nutriment')" class="yuecai-dropdown-item">
                                    <span class="dropdown-icon">📊</span>我的饮食
                                </el-dropdown-item>
                            </el-dropdown-menu>
                        </el-dropdown>
                    </li>
                    <li class="nav-item dropdown-item">
                        <el-dropdown type="success" size="mini" :hide-on-click="false" class="yuecai-dropdown">
                            <span class="el-dropdown-link yuecai-dropdown-trigger">
                                <span class="nav-icon">🎨</span>
                                <span class="nav-text">内容创作</span>
                                <i class="el-icon-arrow-down el-icon--right"></i>
                            </span>
                            <el-dropdown-menu slot="dropdown" class="yuecai-dropdown-menu">
                                <el-dropdown-item @click.native="route('/createGourmet')" class="yuecai-dropdown-item">
                                    <span class="dropdown-icon">🍽️</span>发布美食帖子
                                </el-dropdown-item>
                                <el-dropdown-item @click.native="route('/myCookbook')" class="yuecai-dropdown-item">
                                    <span class="dropdown-icon">📒</span>我的食谱
                                </el-dropdown-item>
                            </el-dropdown-menu>
                        </el-dropdown>
                    </li>
                </ul>
                
                <div class="right-actions">
                    <el-button 
                        type="primary" 
                        size="mini" 
                        @click="route('/service')" 
                        class="yuecai-button"
                        round>
                        <span class="button-icon"></span>内容中心
                    </el-button>
                    
                    <el-dropdown type="success" size="mini" class="user-dropdown yuecai-dropdown">
                        <span class="el-dropdown-link user-dropdown-trigger" style="display: flex; align-items: center;cursor: pointer;">
                            <div class="user-avatar-wrapper">
                                <el-avatar :size="30" :src="userInfo.userAvatar" style="margin-top: 0;"></el-avatar>
                                <div class="avatar-ring"></div>
                            </div>
                            <span class="user-name" style="margin-left: 8px;font-size: 14px;">{{ userInfo.userName }}</span>
                            <i class="el-icon-arrow-down el-icon--right" style="margin-left: 5px;"></i>
                        </span>
                        <el-dropdown-menu slot="dropdown" class="yuecai-dropdown-menu user-menu">
                            <el-dropdown-item @click.native="route('/self')" class="yuecai-dropdown-item">
                                <span class="dropdown-icon">👤</span>个人资料
                            </el-dropdown-item>
                            <el-dropdown-item @click.native="route('/resetPwd')" class="yuecai-dropdown-item">
                                <span class="dropdown-icon">🔐</span>修改密码
                            </el-dropdown-item>
                            <el-dropdown-item @click.native="loginOut" class="yuecai-dropdown-item logout-item">
                                <span class="dropdown-icon">🚪</span>退出登录
                            </el-dropdown-item>
                        </el-dropdown-menu>
                    </el-dropdown>
                </div>
            </div>
        </div>
        
        <div class="router-view">
            <!-- 添加装饰性边框 -->
            <div class="router-view-border-top"></div>
            <div class="router-view-content">
                <router-view></router-view>
            </div>
        </div>
    </div>
</template>

<script>
import Logo from "@/components/Logo"
export default {
    components: { Logo },
    name: "User",
    data() {
        return {
            key: '',
            defaultPath: '/gourmet',
            userInfo: {}
        }
    },
    created() {
        this.auth();
        // 默认加载景点页
        this.route(this.defaultPath);
    },
    methods: {
        // 路由跳转
        route(path) {
            if (this.$route.path !== path) {
                this.$router.push(path);
            };
        },
        // 退出登录
        async loginOut() {
            const confirmed = await this.$swalConfirm({
                title: '退出登录',
                text: `退出后需要重新登录哦？`,
                icon: 'warning',
            });
            if (confirmed) {
                sessionStorage.setItem('token', null);
                this.$router.push('/');
            }
        },
        // Token 检验
        async auth() {
            const { data } = await this.$axios.get('/user/auth');
            if (data.code !== 200) { // Token校验异常
                this.$router.push('/');
            } else {
                this.userInfo = data.data;
                // 存储用户信息
                sessionStorage.setItem('userInfo', JSON.stringify(this.userInfo));
            }
        },
    }
};
</script>

<style scoped lang="scss">
// 粤菜主题色彩方案
$yuecai-primary: #D4A017; // 金色，代表粤菜的精緻和鲜美
$yuecai-secondary: #E67E22; // 橙色，代表粤菜的温暖和热情
$yuecai-accent: #27AE60; // 绿色，代表新鲜食材
$yuecai-light: #FFF9E6; // 浅米色，代表米饭菜香
$yuecai-dark: #8B4513; // 深棕色，代表酱油和酱料
$yuecai-bg: #FFFEF9; // 页面背景色
$yuecai-nav-hover: #F1C40F; // 悬停金色

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.yuecai-container {
    position: relative;
    min-height: 100vh;
    background-color: $yuecai-bg;
    background-image: 
        radial-gradient(circle at 10% 20%, rgba(255, 245, 215, 0.3) 0%, transparent 20%),
        radial-gradient(circle at 90% 80%, rgba(255, 235, 205, 0.2) 0%, transparent 20%);
    overflow-x: hidden; /* 防止横向滚动 */
}

.decorative-top {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 4px;
    background: linear-gradient(90deg, 
        $yuecai-primary, 
        $yuecai-secondary, 
        $yuecai-accent, 
        $yuecai-primary);
    z-index: 3000;
}

.top {
    height: 70px;
    min-height: 70px;
    position: sticky;
    top: 0;
    background: linear-gradient(135deg, rgba(255, 255, 255, 0.95) 0%, rgba(255, 249, 230, 0.95) 100%);
    z-index: 2000;
    padding: 0 20px;
    box-sizing: border-box;
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid rgba(212, 160, 23, 0.2);
    box-shadow: 0 4px 20px rgba(212, 160, 23, 0.1);
    backdrop-filter: blur(10px);
    
    .top-bg-pattern {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        opacity: 0.05;
        background-image: url("data:image/svg+xml,%3Csvg width='60' height='60' viewBox='0 0 60 60' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='none' fill-rule='evenodd'%3E%3Cg fill='%23D4A017' fill-opacity='0.4'%3E%3Cpath d='M36 34v-4h-2v4h-4v2h4v4h2v-4h4v-2h-4zm0-30V0h-2v4h-4v2h4v4h2V6h4V4h-4zM6 34v-4H4v4H0v2h4v4h2v-4h4v-2H6zM6 4V0H4v4H0v2h4v4h2V6h4V4H6z'/%3E%3C/g%3E%3C/g%3E%3C/svg%3E");
        z-index: -1;
    }

    .top-left {
        display: flex;
        align-items: center;
        flex-shrink: 0;
        min-width: 200px;
        
        .logo-decoration {
            margin-left: 10px;
            display: flex;
            gap: 5px;
            
            .decoration-icon {
                font-size: 20px;
                animation: float 3s ease-in-out infinite;
                
                &:nth-child(2) {
                    animation-delay: 0.5s;
                }
            }
        }
    }

    .top-right {
        display: flex;
        align-items: center;
        justify-content: flex-end;
        flex: 1;
        gap: 10px;
        
        .nav-menu {
            list-style: none;
            display: flex;
            align-items: center;
            margin: 0;
            padding: 0;
            flex-wrap: nowrap;
            overflow: visible;
            
            .nav-item {
                cursor: pointer;
                margin: 0 4px;
                padding: 8px 12px;
                min-width: 60px;
                color: $yuecai-dark;
                font-size: 14px;
                box-sizing: border-box;
                font-weight: 500;
                border-radius: 8px;
                transition: all 0.3s ease;
                position: relative;
                display: flex;
                align-items: center;
                white-space: nowrap;
                height: 40px;
                
                .nav-icon {
                    margin-right: 6px;
                    font-size: 16px;
                    flex-shrink: 0;
                }
                
                .nav-text {
                    overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                }
                
                &:hover {
                    color: $yuecai-secondary;
                    background-color: rgba(212, 160, 23, 0.1);
                    transform: translateY(-2px);
                    
                    &::after {
                        content: '';
                        position: absolute;
                        bottom: 0;
                        left: 50%;
                        transform: translateX(-50%);
                        width: 20px;
                        height: 3px;
                        background-color: $yuecai-primary;
                        border-radius: 3px;
                    }
                }
                
                &:active {
                    transform: translateY(0);
                }
            }
            
            .dropdown-item {
                padding: 0;
                
                .yuecai-dropdown-trigger {
                    display: flex;
                    align-items: center;
                    padding: 8px 12px;
                    height: 100%;
                    color: $yuecai-dark;
                    font-size: 14px;
                    font-weight: 500;
                    
                    &:hover {
                        color: $yuecai-secondary;
                    }
                }
            }
        }
        
        .right-actions {
            display: flex;
            align-items: center;
            gap: 15px;
            margin-left: 10px;
            flex-shrink: 0;
        }
    }
}

.yuecai-button {
    background: linear-gradient(135deg, $yuecai-primary 0%, $yuecai-secondary 100%);
    border: none;
    box-shadow: 0 4px 10px rgba(212, 160, 23, 0.3);
    transition: all 0.3s ease;
    font-weight: 600;
    white-space: nowrap;
    
    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 15px rgba(212, 160, 23, 0.4);
        background: linear-gradient(135deg, lighten($yuecai-primary, 5%) 0%, lighten($yuecai-secondary, 5%) 100%);
    }
    
    &:active {
        transform: translateY(0);
    }
    
    .button-icon {
        margin-right: 5px;
    }
}

.yuecai-dropdown {
    .yuecai-dropdown-trigger {
        color: $yuecai-dark;
        font-weight: 500;
        padding: 8px 12px;
        border-radius: 8px;
        transition: all 0.3s ease;
        display: flex;
        align-items: center;
        
        &:hover {
            background-color: rgba(212, 160, 23, 0.1);
            color: $yuecai-secondary;
        }
    }
    
    .yuecai-dropdown-menu {
        border: 1px solid rgba(212, 160, 23, 0.2);
        border-radius: 8px;
        box-shadow: 0 5px 20px rgba(212, 160, 23, 0.15);
        overflow: hidden;
        
        .yuecai-dropdown-item {
            padding: 10px 15px;
            color: $yuecai-dark;
            transition: all 0.2s ease;
            display: flex;
            align-items: center;
            
            .dropdown-icon {
                margin-right: 8px;
            }
            
            &:hover {
                background-color: rgba(212, 160, 23, 0.1);
                color: $yuecai-secondary;
                padding-left: 20px;
            }
        }
        
        .logout-item:hover {
            color: #E74C3C;
            background-color: rgba(231, 76, 60, 0.1);
        }
    }
}

.user-dropdown-trigger {
    padding: 5px 10px;
    border-radius: 20px;
    transition: all 0.3s ease;
    
    &:hover {
        background-color: rgba(212, 160, 23, 0.1);
    }
}

.user-avatar-wrapper {
    position: relative;
    width: 30px;
    height: 30px;
    
    .avatar-ring {
        position: absolute;
        top: -3px;
        left: -3px;
        right: -3px;
        bottom: -3px;
        border: 2px solid transparent;
        border-radius: 50%;
        background: linear-gradient(135deg, $yuecai-primary, $yuecai-accent) border-box;
        -webkit-mask: 
            linear-gradient(#fff 0 0) padding-box, 
            linear-gradient(#fff 0 0);
        -webkit-mask-composite: destination-out;
        mask: 
            linear-gradient(#fff 0 0) padding-box, 
            linear-gradient(#fff 0 0);
        mask-composite: exclude;
        -webkit-mask-composite: destination-out;
        animation: rotate 10s linear infinite;
    }
}

.router-view {
    padding: 20px;
    box-sizing: border-box;
    min-height: calc(100vh - 70px);
    position: relative;
    overflow-x: hidden;
    
    .router-view-border-top {
        position: absolute;
        top: 0;
        left: 5%;
        width: 90%;
        height: 1px;
        background: linear-gradient(90deg, 
            transparent, 
            rgba(212, 160, 23, 0.3), 
            transparent);
    }
    
    .router-view-content {
        width: 100%;
        max-width: 1200px;
        margin: 0 auto;
        background-color: white;
        border-radius: 12px;
        box-shadow: 0 5px 20px rgba(212, 160, 23, 0.08);
        padding: 20px;
        min-height: calc(100vh - 120px);
        position: relative;
        overflow: hidden;
        
        &::before {
            content: '';
            position: absolute;
            top: 0;
            right: 0;
            width: 200px;
            height: 200px;
            background: radial-gradient(circle, rgba(212, 160, 23, 0.05) 0%, transparent 70%);
            z-index: 0;
        }
        
        &::after {
            content: '';
            position: absolute;
            bottom: 0;
            left: 0;
            width: 150px;
            height: 150px;
            background: radial-gradient(circle, rgba(39, 174, 96, 0.05) 0%, transparent 70%);
            z-index: 0;
        }
        
        & > * {
            position: relative;
            z-index: 1;
        }
    }
}

// 动画效果
@keyframes float {
    0%, 100% {
        transform: translateY(0);
    }
    50% {
        transform: translateY(-5px);
    }
}

@keyframes rotate {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
}

// 响应式调整
@media screen and (max-width: 1200px) {
    .top {
        padding: 0 15px;
        
        .nav-menu .nav-item {
            margin: 0 3px;
            padding: 6px 8px;
            font-size: 13px;
            
            .nav-icon {
                font-size: 14px;
                margin-right: 4px;
            }
        }
    }
}

@media screen and (max-width: 992px) {
    .top {
        flex-wrap: wrap;
        height: auto;
        min-height: auto;
        padding: 10px 15px;
        
        .top-left {
            width: 100%;
            justify-content: center;
            margin-bottom: 10px;
        }
        
        .top-right {
            width: 100%;
            justify-content: center;
            flex-direction: column;
            
            .nav-menu {
                justify-content: center;
                flex-wrap: wrap;
                margin-bottom: 10px;
                
                .nav-item {
                    margin: 2px 4px;
                }
            }
            
            .right-actions {
                width: 100%;
                justify-content: center;
            }
        }
    }
    
    .router-view {
        padding: 10px;
        
        .router-view-content {
            padding: 15px;
        }
    }
}

@media screen and (max-width: 768px) {
    .top {
        padding: 8px 10px;
        
        .top-left {
            margin-bottom: 8px;
            
            .logo-decoration {
                display: none;
            }
        }
        
        .top-right {
            .nav-menu {
                .nav-item {
                    margin: 2px;
                    padding: 4px 6px;
                    font-size: 12px;
                    
                    .nav-icon {
                        font-size: 12px;
                        margin-right: 2px;
                    }
                }
            }
            
            .right-actions {
                flex-direction: column;
                gap: 8px;
                
                .yuecai-button {
                    width: 100%;
                    justify-content: center;
                }
            }
        }
    }
    
    .router-view {
        padding: 8px;
        
        .router-view-content {
            padding: 10px;
            border-radius: 8px;
        }
    }
}
</style>