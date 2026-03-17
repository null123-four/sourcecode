<template>
  <!-- 移除导航栏，设置为全屏帖子内容 -->
  <div class="gourmet-fullpage">
    <!-- 帖子内容容器 -->
    <div class="gourmet-main-content">
      <div class="gourmet-header">
        <!-- 返回按钮 -->
        <div class="back-button" @click="goBack">
          <i class="el-icon-arrow-left"></i> 返回
        </div>
        
        <!-- 帖子标题和基本信息 -->
        <div class="gourmet-meta">
          <div class="title">{{ gourmet.title }}</div>
          <div class="info">
            <img style="width: 20px;height: 20px;border-radius: 10px;" :src="gourmet.userAvatar" alt="">
            <span>{{ gourmet.userName }}</span>
            <span>{{ gourmet.createTime }}</span>
            <span>点赞({{ gourmet.upvoteCount }})</span>
            <span>浏览({{ gourmet.viewCount }})</span>
            <span>收藏({{ gourmet.saveCount }})</span>
          </div>
        </div>
      </div>

      <!-- 帖子正文区域 -->
      <div class="gourmet-body">
        <div class="content-wrapper">
          <div v-html="gourmet.content" class="gourmet-content"></div>
        </div>

        <!-- 点赞收藏操作区域 -->
        <div class="gourmet-actions">
          <span>
            <span class="upvote-operation">
              <el-tooltip class="item" effect="dark" :content="upvoteStatus ? '取消点赞' : '点赞这篇帖子'" placement="bottom">
                <span @click="opeationUpvote">
                  <i class="el-icon-discount"></i>
                  <span>{{ gourmet.upvoteCount }}人喜欢</span>
                </span>
              </el-tooltip>
            </span>
            <span class="upvote-operation">
              <el-tooltip class="item" effect="dark" :content="saveStatus ? '取消收藏' : '收藏这篇帖子'" placement="bottom">
                <span @click="opeationSave">
                  <i class="el-icon-star-off"></i>
                  <span>{{ gourmet.saveCount }}人收藏</span>
                </span>
              </el-tooltip>
            </span>
          </span>
        </div>
      </div>
    </div>

    <!-- 侧边栏（评分信息） -->
    <div class="gourmet-sidebar">
      <!-- 评分区域 -->
      <div class="rating-section">
        <h4>评分信息</h4>
        
        <!-- 如果用户未评分，显示评分区域 -->
        <div v-if="!ratingStatus" class="rate-input-area">
          <div class="rate-hint">轻触评个分数吧</div>
          <el-rate @change="ratingEvent" v-model="ratingScore" show-text></el-rate>
        </div>
        
        <!-- 总评分显示 -->
        <div class="total-rating">
          <div class="rating-title">总评分</div>
          <el-rate v-model="gourmet.rating" disabled show-score text-color="#ff9900" score-template="{value}">
          </el-rate>
        </div>
        
        <!-- 用户评分列表 -->
        <div class="user-ratings">
          <div v-if="ratingVos.length === 0" class="no-rating">
            <el-empty description="暂无评论哦！"></el-empty>
          </div>
          <div v-else v-for="(ratingVO, m) in ratingVos" :key="m" class="rating-item">
            <div class="rating-user">
              <img style="width: 20px;height: 20px;border-radius: 10px;" :src="ratingVO.userAvatar" alt="" srcset="">
              <span>{{ ratingVO.userName }}</span>
            </div>
            <div class="rating-score">
              <el-rate v-model="ratingVO.score" disabled show-score text-color="#ff9900" score-template="{value}">
              </el-rate>
            </div>
            <div class="rating-time">
              {{ ratingVO.createTime }}
            </div>
          </div>
        </div>
      </div>

      <!-- 评论区 -->
      <div class="comments-section">
        <h4>评论与讨论</h4>
        <Evaluations :contentId="gourmetId" contentType="GOURMET" />
      </div>
    </div>
  </div>
</template>

<script>
import Evaluations from "@/components/Evaluations"
export default {
  components: { Evaluations },
  name: "GourmetDetail",
  data() {
    return {
      gourmetId: null,
      gourmet: {},
      ratingScore: 0, // 评分
      recommendGourmet: [],
      ratingVos: [], // 用户评分数据
      upvoteStatus: false, // 默认未点赞
      saveStatus: false, // 默认未收藏
      ratingStatus: false, // 评分状态
    }
  },
  created() {
    this.loadGourmetId();
    this.fetchTopGourmet();
  },
  methods: {
    // 返回上一页
    goBack() {
      this.$router.go(-1);
    },
    // 查询用户的评分数据
    fetchRatingData(contentId) {
      const interactionQueryDto = {
        contentId,
        current: 1,
        size: 3
      }
      this.$axios.post(`/interaction/queryUserRating`, interactionQueryDto).then(res => {
        const { data } = res;
        if (data.code === 200) {
          this.ratingVos = data.data;
        }
      }).catch(error => {
        console.log("查询评分信息异常：", error);
      })
    },
    // 用户评分时触发
    ratingEvent() {
      this.$axios.post(`/interaction/ratingOperation/${this.gourmetId}/${this.ratingScore}`).then(res => {
        const { data } = res;
        if (data.code === 200) {
          this.gourmet = data.data[0];
          this.dealRating(this.gourmet);
          this.$notify({
            duration: 1000, // 毫秒
            title: '评分',
            message: '评分成功',
            type: 'success'
          });
          this.ratingStatus = true;
          this.fetchRatingData(this.gourmetId);
        } else {
          this.$message(data.msg);
        }
      }).catch(error => {
        console.log("评分操作异常：", error);
      })
    },
    // 查询点赞状态
    fetchUpvoteOperation(contentId) {
      this.$axios.get(`/interaction/upvoteStatus/${contentId}`).then(res => {
        const { data } = res;
        if (data.code === 200) {
          console.log("点赞的状态=>", data.data);
          this.upvoteStatus = data.data > 0;
        }
      }).catch(error => {
        console.log("通过点赞状态异常：", error);
      })
    },
    // 查询评分状态
    fetchRatingOperation(contentId) {
      this.$axios.get(`/interaction/ratingStatus/${contentId}`).then(res => {
        const { data } = res;
        if (data.code === 200) {
          console.log("评分的状态=>", data.data);
          this.ratingStatus = data.data > 0;
        }
      }).catch(error => {
        console.log("通过评分状态异常：", error);
      })
    },
    // 查询收藏状态
    fetchSaveOperation(contentId) {
      this.$axios.get(`/interaction/saveStatus/${contentId}`).then(res => {
        const { data } = res;
        if (data.code === 200) {
          console.log("收藏的状态=>", data.data);
          this.saveStatus = data.data > 0;
        }
      }).catch(error => {
        console.log("通过收藏状态异常：", error);
      })
    },
    // 收藏操作
    opeationSave() {
      this.$axios.post(`/interaction/saveOperation/${this.gourmetId}`).then(res => {
        const { data } = res;
        if (data.code === 200) {
          console.log(data.data > 0 ? '收藏成功' : '取消收藏成功');
          if (data.data > 0) {
            this.gourmet.saveCount += 1;
          } else {
            this.gourmet.saveCount -= 1;
          }
          this.saveStatus = data.data > 0;
        }
      }).catch(error => {
        console.log("收藏操作异常：", error);
      })
    },
    // 点赞操作
    opeationUpvote() {
      this.$axios.post(`/interaction/upvoteOperation/${this.gourmetId}`).then(res => {
        const { data } = res;
        if (data.code === 200) {
          console.log(data.data > 0 ? '点赞成功' : '取消点赞成功');
          if (data.data > 0) {
            this.gourmet.upvoteCount += 1;
          } else {
            this.gourmet.upvoteCount -= 1;
          }
          this.upvoteStatus = data.data > 0;
        }
      }).catch(error => {
        console.log("点赞操作异常：", error);
      })
    },
    readGourmet(gourmet) {
      this.fetchGourmetById(gourmet.id);
    },
    loadGourmetId() {
      this.gourmetId = Number(sessionStorage.getItem('gourmetId'));
      // 加载美食做法数据
      this.fetchGourmetById(this.gourmetId);
      // 处理浏览操作
      this.viewOperation(this.gourmetId);
      // 加载点赞状态
      this.fetchUpvoteOperation(this.gourmetId);
      // 加载收藏状态
      this.fetchSaveOperation(this.gourmetId);
      // 加载评分状态
      this.fetchRatingOperation(this.gourmetId);
      // 加载用户的评分信息
      this.fetchRatingData(this.gourmetId);
    },
    dealRating(gourmet) {
      this.gourmet.rating = gourmet.rating === null ? 0 : gourmet.rating;
    },
    // 通过ID查找对应的美食做法
    fetchGourmetById(gourmetId) {
      this.$axios.get(`/gourmet/${gourmetId}`).then(res => {
        const { data } = res;
        if (data.code === 200) {
          this.gourmet = data.data[0];
          this.dealRating(this.gourmet);
        }
      }).catch(error => {
        console.log("通过ID查询美食做法异常：", error);
      })
    },
    // 浏览操作
    viewOperation(contentId) {
      this.$axios.post(`/interaction/viewOperation/${contentId}`).then(res => {
        const { data } = res;
        if (data.code === 200) {
          console.log("浏览执行...");
        }
      }).catch(error => {
        console.log("浏览操作异常：", error);
      })
    },
    // 获取通过算法查回来，现在只是占位处理
    fetchTopGourmet() {
      this.$axios.post(`/gourmet/queryList`, { size: 3, current: 1 }).then(res => {
        const { data } = res;
        if (data.code === 200) {
          this.recommendGourmet = data.data;
        }
      }).catch(error => {
        console.log("查询推荐美食做法异常：", error);
      })
    },
  }
};
</script>

<style scoped lang="scss">
// 粤菜主题色彩
$yuecai-primary: #D4A017; // 金色
$yuecai-secondary: #E67E22; // 橙色
$yuecai-accent: #27AE60; // 绿色
$yuecai-light: #FFF9E6; // 浅米色
$yuecai-dark: #8B4513; // 深棕色
$yuecai-bg: #FFFEF9; // 页面背景色

.gourmet-fullpage {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  background-color: $yuecai-bg;
  display: flex;
  overflow: hidden;
  z-index: 1000;
}

.gourmet-main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  background-color: white;
  overflow: hidden;
  position: relative;
}

.gourmet-header {
  padding: 20px 40px;
  border-bottom: 1px solid rgba(212, 160, 23, 0.1);
  background: linear-gradient(to right, rgba(255, 249, 230, 0.8), rgba(255, 250, 240, 0.8));
  min-height: 120px;
  position: relative;
  box-shadow: 0 2px 10px rgba(212, 160, 23, 0.1);
  
  .back-button {
    display: inline-flex;
    align-items: center;
    padding: 8px 16px;
    background: rgba(212, 160, 23, 0.1);
    color: $yuecai-dark;
    border-radius: 20px;
    cursor: pointer;
    font-size: 14px;
    font-weight: 500;
    transition: all 0.3s ease;
    margin-bottom: 20px;
    border: 1px solid rgba(212, 160, 23, 0.2);
    
    &:hover {
      background: rgba(212, 160, 23, 0.2);
      transform: translateX(-5px);
    }
    
    i {
      margin-right: 5px;
    }
  }
  
  .gourmet-meta {
    .title {
      font-size: 32px;
      font-weight: 800;
      margin-block: 6px;
      color: $yuecai-dark;
      line-height: 1.3;
      background: linear-gradient(135deg, $yuecai-primary, $yuecai-secondary);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      background-clip: text;
    }
    
    .info {
      font-size: 14px;
      padding: 4px 6px;
      color: rgba(90, 89, 89, 0.8);
      display: flex;
      justify-content: flex-start;
      align-items: center;
      gap: 20px;
      margin-block: 4px;
      flex-wrap: wrap;
      
      img {
        width: 24px;
        height: 24px;
        border-radius: 12px;
        border: 2px solid rgba(212, 160, 23, 0.3);
      }
      
      span {
        display: flex;
        align-items: center;
        gap: 5px;
      }
      
      span:not(:first-child) {
        padding-left: 20px;
        position: relative;
        
        &::before {
          content: '';
          position: absolute;
          left: 0;
          top: 50%;
          transform: translateY(-50%);
          width: 1px;
          height: 12px;
          background-color: rgba(90, 89, 89, 0.2);
        }
      }
    }
  }
}

.gourmet-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  padding: 0 40px;
  background: linear-gradient(135deg, rgba(255, 249, 230, 0.1) 0%, rgba(255, 255, 255, 1) 100%);
  
  .content-wrapper {
    flex: 1;
    overflow-y: auto;
    padding: 20px 0;
    margin: 20px 0;
    
    &::-webkit-scrollbar {
      width: 8px;
    }
    
    &::-webkit-scrollbar-track {
      background: rgba(212, 160, 23, 0.05);
      border-radius: 4px;
    }
    
    &::-webkit-scrollbar-thumb {
      background: linear-gradient(135deg, $yuecai-primary 0%, $yuecai-secondary 100%);
      border-radius: 4px;
      
      &:hover {
        background: linear-gradient(135deg, lighten($yuecai-primary, 10%) 0%, lighten($yuecai-secondary, 10%) 100%);
      }
    }
    
    .gourmet-content {
      line-height: 1.8;
      font-size: 16px;
      color: #333;
      max-width: 900px;
      margin: 0 auto;
      
      // 设置内容样式
      img {
        max-width: 100%;
        height: auto;
        border-radius: 8px;
        margin: 20px 0;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
      }
      
      p {
        margin-bottom: 1.5em;
      }
      
      h2, h3, h4 {
        color: $yuecai-dark;
        margin-top: 2em;
        margin-bottom: 1em;
        padding-bottom: 8px;
        border-bottom: 2px solid rgba(212, 160, 23, 0.2);
      }
    }
  }
  
  .gourmet-actions {
    padding: 20px 0;
    border-top: 1px solid rgba(212, 160, 23, 0.1);
    display: flex;
    justify-content: center;
    align-items: center;
    background: rgba(255, 249, 230, 0.5);
    border-radius: 12px 12px 0 0;
    margin-top: auto;
  }
}

.gourmet-sidebar {
  width: 400px;
  background: linear-gradient(to bottom, rgba(255, 250, 240, 0.9), rgba(255, 252, 245, 0.9));
  border-left: 1px solid rgba(212, 160, 23, 0.1);
  overflow-y: auto;
  padding: 20px;
  box-shadow: -5px 0 20px rgba(212, 160, 23, 0.1);
  position: relative;
  
  &::-webkit-scrollbar {
    width: 6px;
  }
  
  &::-webkit-scrollbar-track {
    background: rgba(212, 160, 23, 0.05);
  }
  
  &::-webkit-scrollbar-thumb {
    background: rgba(212, 160, 23, 0.2);
    border-radius: 3px;
    
    &:hover {
      background: rgba(212, 160, 23, 0.3);
    }
  }
  
  h4 {
    color: $yuecai-dark;
    font-size: 20px;
    margin-bottom: 20px;
    padding-bottom: 10px;
    border-bottom: 2px solid rgba(212, 160, 23, 0.3);
  }
  
  .rating-section {
    background: white;
    border-radius: 12px;
    padding: 20px;
    margin-bottom: 30px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
    
    .rate-input-area {
      margin: 20px 0;
      text-align: center;
      
      .rate-hint {
        margin-bottom: 10px;
        font-size: 12px;
        color: rgba(4, 81, 165, 0.8);
      }
    }
    
    .total-rating {
      background: linear-gradient(135deg, rgba(255, 249, 230, 0.8), rgba(255, 245, 220, 0.8));
      border-radius: 8px;
      padding: 20px;
      margin: 20px 0;
      
      .rating-title {
        text-align: center;
        font-weight: bold;
        font-size: 18px;
        margin-bottom: 10px;
        color: $yuecai-dark;
      }
    }
    
    .user-ratings {
      .rating-item {
        padding: 15px 0;
        border-bottom: 1px solid rgba(212, 160, 23, 0.1);
        
        &:last-child {
          border-bottom: none;
        }
        
        .rating-user {
          display: flex;
          align-items: center;
          gap: 8px;
          margin-bottom: 8px;
          
          img {
            width: 24px;
            height: 24px;
            border-radius: 12px;
            border: 2px solid rgba(212, 160, 23, 0.3);
          }
          
          span {
            font-size: 12px;
            font-weight: 500;
            color: $yuecai-dark;
          }
        }
        
        .rating-score {
          margin: 8px 0;
        }
        
        .rating-time {
          font-size: 12px;
          color: rgba(51, 51, 51, 0.7);
        }
      }
      
      .no-rating {
        text-align: center;
        padding: 30px 0;
      }
    }
  }
  
  .comments-section {
    background: white;
    border-radius: 12px;
    padding: 20px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  }
}

.upvote-operation {
  font-weight: 600;
  font-size: 16px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  border-radius: 8px;
  background: white;
  border: 1px solid rgba(212, 160, 23, 0.2);
  margin: 0 10px;
  transition: all 0.3s ease;
  color: $yuecai-dark;
  
  i {
    color: $yuecai-secondary;
    font-size: 18px;
  }
  
  &:hover {
    background: linear-gradient(135deg, rgba(212, 160, 23, 0.1), rgba(230, 126, 34, 0.1));
    border-color: $yuecai-primary;
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(212, 160, 23, 0.2);
  }
  
  &:active {
    transform: translateY(0);
  }
}

// 响应式设计
@media screen and (max-width: 1200px) {
  .gourmet-sidebar {
    width: 350px;
  }
  
  .gourmet-header,
  .gourmet-body {
    padding-left: 30px;
    padding-right: 30px;
  }
}

@media screen and (max-width: 992px) {
  .gourmet-fullpage {
    flex-direction: column;
  }
  
  .gourmet-sidebar {
    width: 100%;
    border-left: none;
    border-top: 1px solid rgba(212, 160, 23, 0.1);
    max-height: 50vh;
  }
  
  .gourmet-header {
    padding: 20px;
  }
  
  .gourmet-body {
    padding: 0 20px;
  }
}

@media screen and (max-width: 768px) {
  .gourmet-header {
    padding: 15px;
    min-height: 100px;
    
    .title {
      font-size: 24px;
    }
    
    .info {
      font-size: 12px;
      gap: 10px;
      
      span:not(:first-child) {
        padding-left: 10px;
      }
    }
  }
  
  .gourmet-body {
    padding: 0 15px;
    
    .content-wrapper {
      padding: 10px 0;
      
      .gourmet-content {
        font-size: 14px;
      }
    }
  }
  
  .gourmet-sidebar {
    padding: 15px;
  }
  
  .upvote-operation {
    padding: 8px 15px;
    font-size: 14px;
  }
}
</style>