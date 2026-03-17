<template>
  <div class="evaluations-container">
    <!-- 评论头部 - 移除了评论计数 -->
    <div class="comment-header">
      <h2 class="comment-title">评论与讨论</h2>
    </div>

    <!-- 评论输入区域 -->
    <div class="comment-input-section">
      <div class="comment-input-wrapper">
        <el-avatar 
          :size="40" 
          :src="userData.userAvatar" 
          class="user-avatar"
          @error="handleAvatarError"
        >
          {{ getUserInitial(userData.userName) }}
        </el-avatar>
        <div class="input-container">
          <div class="input-wrapper">
            <textarea 
              v-model="content" 
              placeholder="请友好交流" 
              maxlength="300"
              class="comment-input"
              rows="3"
            ></textarea>
            <div class="input-footer">
              <el-button 
                type="primary" 
                size="small" 
                :disabled="!content.trim()"
                @click="commentClick"
                class="submit-btn"
              >
                评论
              </el-button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 评论列表 -->
    <div class="comment-list" v-if="commentList.length > 0">
      <div 
        v-for="(comment, index) in commentList" 
        :key="comment.id || index" 
        class="comment-item"
      >
        <!-- 评论主体 -->
        <div class="comment-main">
          <!-- 用户信息 -->
          <div class="user-info">
            <el-avatar 
              :size="36" 
              :src="formatAvatarUrl(comment.userAvatar)" 
              class="comment-avatar"
              @error="handleCommentAvatarError(comment)"
            >
              {{ getUserInitial(comment.userName) }}
            </el-avatar>
            <div class="user-details">
              <div class="user-name-row">
                <span class="user-name">{{ comment.userName }}</span>
                <span v-if="comment.userId == userId" class="self-tag">我自己</span>
                <span class="comment-time">{{ comment.time }}</span>
              </div>
            </div>
          </div>

          <!-- 评论内容 -->
          <div class="comment-content">
            {{ comment.content }}
          </div>

          <!-- 操作按钮 -->
          <div class="comment-actions">
            <span 
              class="action-item" 
              :class="{ active: comment.upvoteFlag }"
              @click="upvote(comment)"
            >
              <i class="el-icon-thumb"></i>
              <span>{{ comment.upvoteCount || 0 }}</span>
            </span>
            
            <span 
              class="action-item" 
              @click="toggleReplyInput(comment)"
            >
              <i class="el-icon-chat-dot-round"></i>
              <span>回复</span>
              <span v-if="comment.childTotal > 0" class="reply-count">({{ comment.childTotal }})</span>
            </span>
            
            <span 
              v-if="comment.userId == userId" 
              class="action-item delete-btn" 
              @click="confirmDelete(comment)"
            >
              <i class="el-icon-delete"></i>
              <span>删除</span>
            </span>
          </div>

          <!-- 回复输入框（父级评论） -->
          <div v-if="comment.showReplyInput" class="reply-input-wrapper">
            <div class="input-wrapper">
              <textarea 
                v-model="replyContent" 
                placeholder="我要回复" 
                maxlength="300"
                class="comment-input"
                rows="2"
              ></textarea>
              <div class="input-footer">
                <el-button 
                  type="primary" 
                  size="small" 
                  :disabled="!replyContent.trim()"
                  @click="submitReply(comment)"
                  class="submit-btn"
                >
                  评论
                </el-button>
              </div>
            </div>
          </div>
        </div>

        <!-- 子级评论 -->
        <div v-if="comment.commentChildVOS && comment.commentChildVOS.length > 0" class="child-comments">
          <div 
            v-for="child in comment.commentChildVOS" 
            :key="child.id" 
            class="child-comment-item"
          >
            <!-- 子评论用户信息 -->
            <div class="child-user-info">
              <el-avatar 
                :size="28" 
                :src="formatAvatarUrl(child.userAvatar)" 
                class="child-comment-avatar"
                @error="handleCommentAvatarError(child)"
              >
                {{ getUserInitial(child.userName) }}
              </el-avatar>
              <div class="child-user-details">
                <div class="child-user-name-row">
                  <span class="child-user-name">{{ child.userName }}</span>
                  <span v-if="child.userId == userId" class="self-tag">我自己</span>
                  <span v-if="child.replierName" class="reply-to">
                    <span class="reply-label">回复</span>
                    <el-avatar 
                      :size="16" 
                      :src="formatAvatarUrl(child.replierAvatar)" 
                      class="replier-avatar"
                      @error="handleReplierAvatarError(child)"
                    >
                      {{ getUserInitial(child.replierName) }}
                    </el-avatar>
                    <span class="replier-name">{{ child.replierName }}</span>
                    <span v-if="child.replierId == userId" class="self-tag small">我自己</span>
                  </span>
                  <span class="child-comment-time">{{ child.time }}</span>
                </div>
              </div>
            </div>

            <!-- 子评论内容 -->
            <div class="child-comment-content">
              {{ child.content }}
            </div>

            <!-- 子评论操作 -->
            <div class="child-comment-actions">
              <span 
                class="action-item" 
                :class="{ active: child.upvoteFlag }"
                @click="upvote(child)"
              >
                <i class="el-icon-thumb"></i>
                <span>{{ child.upvoteCount || 0 }}</span>
              </span>
              
              <span 
                class="action-item" 
                @click="toggleChildReplyInput(child)"
              >
                <i class="el-icon-chat-dot-round"></i>
                <span>回复</span>
              </span>
              
              <span 
                v-if="child.userId == userId" 
                class="action-item delete-btn" 
                @click="confirmDelete(child)"
              >
                <i class="el-icon-delete"></i>
                <span>删除</span>
              </span>
            </div>

            <!-- 子评论回复输入框 -->
            <div v-if="child.replyInputStatus" class="child-reply-input-wrapper">
              <div class="input-wrapper">
                <textarea 
                  v-model="replyChildContent" 
                  placeholder="我要回复" 
                  maxlength="300"
                  class="comment-input"
                  rows="2"
                ></textarea>
                <div class="input-footer">
                  <el-button 
                    type="primary" 
                    size="small" 
                    :disabled="!replyChildContent.trim()"
                    @click="submitChildReply(child)"
                    class="submit-btn"
                  >
                    评论
                  </el-button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 空状态 -->
    <div v-else class="empty-state">
      <el-empty description="暂无评论，快来发表第一条评论吧！"></el-empty>
    </div>

    <!-- 删除确认对话框 -->
    <el-dialog
      title="删除评论"
      :visible.sync="deleteDialogVisible"
      width="400px"
      center
    >
      <span>确定要删除这条评论吗？删除后不可恢复。</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="deleteDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="deleteComment">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { timeAgo } from '@/utils/data'

export default {
  props: {
    contentId: {
      type: Number,
      default: 0
    },
    contentType: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      userData: {},
      content: '',
      commentList: [],
      replyContent: '',
      replyChildContent: '',
      isFocused: false,
      userId: '',
      deleteDialogVisible: false,
      commentToDelete: null
    }
  },
  watch: {
    contentId(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.loadCommentList()
      }
    }
  },
  created() {
    this.getUserInfo()
    this.loadCommentList()
  },
  methods: {
    // 获取用户信息
    getUserInfo() {
      try {
        const userInfo = sessionStorage.getItem('userInfo')
        if (userInfo) {
          this.userData = JSON.parse(userInfo)
          this.userId = this.userData.id
        }
      } catch (error) {
        console.error('解析用户信息失败:', error)
      }
    },

    // 格式化头像URL
    formatAvatarUrl(url) {
      if (!url || url === 'default') {
        return this.generateDefaultAvatar()
      }
      
      // 处理相对路径
      if (url.startsWith('/uploads/')) {
        const baseUrl = process.env.VUE_APP_BASE_API || 'http://localhost:8080'
        return `${baseUrl}${url}`
      }
      
      return url
    },

    // 生成默认头像
    generateDefaultAvatar(name) {
      const colors = [
        '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', 
        '#FFEAA7', '#DDA0DD', '#98D8C8', '#F7DC6F'
      ]
      const char = name ? name.charAt(0).toUpperCase() : 'U'
      const color = colors[char.charCodeAt(0) % colors.length]
      
      return `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100"><circle cx="50" cy="50" r="50" fill="${color}"/><text x="50" y="50" font-family="Arial" font-size="40" fill="white" text-anchor="middle" dy=".35em">${char}</text></svg>`
    },

    // 获取用户名称首字母
    getUserInitial(userName) {
      return userName ? userName.charAt(0).toUpperCase() : 'U'
    },

    // 处理头像加载错误
    handleAvatarError() {
      // 使用默认头像
      return false
    },

    handleCommentAvatarError(comment) {
      comment.userAvatar = this.generateDefaultAvatar(comment.userName)
    },

    handleReplierAvatarError(child) {
      child.replierAvatar = this.generateDefaultAvatar(child.replierName)
    },

    // 点赞/取消点赞
    upvote(comment) {
      this.$axios.post('evaluations/upvoteOperation', { id: comment.id }).then(res => {
        if (res.data.code === 200) {
          const { count, haveUpvote } = res.data.data
          comment.upvoteCount = Number(count)
          comment.upvoteFlag = Boolean(haveUpvote)
        }
      }).catch(err => {
        console.error('点赞操作异常:', err)
        this.$message.error('操作失败，请稍后重试')
      })
    },

    // 确认删除评论
    confirmDelete(comment) {
      this.commentToDelete = comment
      this.deleteDialogVisible = true
    },

    // 删除评论
    deleteComment() {
      if (!this.commentToDelete) return
      
      this.$axios.delete(`evaluations/delete/${this.commentToDelete.id}`).then(res => {
        if (res.data.code === 200) {
          this.$message.success('删除成功')
          this.loadCommentList()
        } else {
          this.$message.error(res.data.msg || '删除失败')
        }
        this.deleteDialogVisible = false
        this.commentToDelete = null
      }).catch(err => {
        console.error('删除评论异常:', err)
        this.$message.error('删除失败，请稍后重试')
        this.deleteDialogVisible = false
        this.commentToDelete = null
      })
    },

    // 输入框聚焦
    onFocus() {
      this.isFocused = true
    },

    // 输入框失焦
    onBlur() {
      if (!this.content.trim()) {
        this.isFocused = false
      }
    },

    // 发表评论
    commentClick() {
      if (!this.content.trim()) {
        this.$message.warning('评论内容不能为空')
        return
      }

      const evaluations = {
        contentType: this.contentType,
        content: this.content,
        contentId: this.contentId
      }

      this.$axios.post('evaluations/insert', evaluations).then(res => {
        if (res.data.code === 200) {
          this.content = ''
          this.isFocused = false
          this.$message.success('评论成功')
          this.loadCommentList()
        } else {
          this.$message.error(res.data.msg || '评论失败')
        }
      }).catch(err => {
        console.error('评论异常:', err)
        this.$message.error('评论失败，请稍后重试')
      })
    },

    // 切换父级评论回复输入框
    toggleReplyInput(comment) {
      this.$set(comment, 'showReplyInput', !comment.showReplyInput)
      if (comment.showReplyInput) {
        this.replyContent = ''
      }
    },

    // 切换子级评论回复输入框
    toggleChildReplyInput(child) {
      this.$set(child, 'replyInputStatus', !child.replyInputStatus)
      if (child.replyInputStatus) {
        this.replyChildContent = ''
      }
    },

    // 提交父级评论的回复
    submitReply(comment) {
      if (!this.replyContent.trim()) {
        this.$message.warning('回复内容不能为空')
        return
      }

      const evaluationsDTO = {
        contentType: this.contentType,
        content: this.replyContent,
        contentId: this.contentId,
        parentId: comment.id
      }

      this.$axios.post('evaluations/insert', evaluationsDTO).then(res => {
        if (res.data.code === 200) {
          this.replyContent = ''
          comment.showReplyInput = false
          this.$message.success('回复成功')
          this.loadCommentList()
        } else {
          this.$message.error(res.data.msg || '回复失败')
        }
      }).catch(err => {
        console.error('回复异常:', err)
        this.$message.error('回复失败，请稍后重试')
      })
    },

    // 提交子级评论的回复
    submitChildReply(child) {
      if (!this.replyChildContent.trim()) {
        this.$message.warning('回复内容不能为空')
        return
      }

      const evaluationsDTO = {
        replierId: child.userId,
        contentType: this.contentType,
        content: this.replyChildContent,
        contentId: this.contentId,
        parentId: child.parentId
      }

      this.$axios.post('evaluations/insert', evaluationsDTO).then(res => {
        if (res.data.code === 200) {
          this.replyChildContent = ''
          child.replyInputStatus = false
          this.$message.success('回复成功')
          this.loadCommentList()
        } else {
          this.$message.error(res.data.msg || '回复失败')
        }
      }).catch(err => {
        console.error('回复异常:', err)
        this.$message.error('回复失败，请稍后重试')
      })
    },

    // 加载评论列表
    loadCommentList() {
      this.$axios.get(`evaluations/list/${this.contentId}/${this.contentType}`).then(res => {
        if (res.data.code === 200) {
          const data = res.data.data
          this.commentList = data.data || []
          
          // 处理时间显示
          this.commentList.forEach(comment => {
            comment.time = timeAgo(comment.createTime)
            if (comment.commentChildVOS) {
              comment.commentChildVOS.forEach(child => {
                child.time = timeAgo(child.createTime)
              })
            }
          })
        }
      }).catch(err => {
        console.error('评论查询异常:', err)
        this.$message.error('加载评论失败')
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.evaluations-container {
  width: 100%;
  background: #fff;
  border-radius: 8px;
  padding: 20px;
  box-sizing: border-box;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

// 评论头部 - 移除了评论计数
.comment-header {
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid #f0f0f0;
  
  .comment-title {
    font-size: 20px;
    font-weight: 600;
    color: #1d2129;
    margin: 0;
  }
  
  // 移除评论计数样式
  // .comment-count {
  //   font-size: 14px;
  //   color: #86909c;
  // }
}

// 评论输入区域
.comment-input-section {
  margin-bottom: 30px;
  
  .comment-input-wrapper {
    display: flex;
    gap: 12px;
    
    .user-avatar {
      flex-shrink: 0;
    }
    
    .input-container {
      flex: 1;
      
      .input-wrapper {
        border: 1px solid #e5e6eb;
        border-radius: 8px;
        background: #f7f8fa;
        overflow: hidden;
        
        .comment-input {
          width: 100%;
          min-height: 80px;
          padding: 12px;
          border: none;
          background: transparent;
          resize: none;
          font-size: 14px;
          line-height: 1.5;
          color: #1d2129;
          outline: none;
          box-sizing: border-box;
          font-family: inherit;
          
          &::placeholder {
            color: #86909c;
          }
        }
        
        .input-footer {
          display: flex;
          justify-content: flex-end;
          padding: 8px 12px;
          border-top: 1px solid #f0f0f0;
          
          .submit-btn {
            font-size: 12px;
            padding: 5px 16px;
            
            &:disabled {
              opacity: 0.5;
              cursor: not-allowed;
            }
          }
        }
      }
    }
  }
}

// 评论列表
.comment-list {
  .comment-item {
    padding: 20px 0;
    border-bottom: 1px solid #f0f0f0;
    
    &:last-child {
      border-bottom: none;
    }
  }
}

// 评论主体
.comment-main {
  .user-info {
    display: flex;
    align-items: center;
    margin-bottom: 8px;
    
    .comment-avatar {
      flex-shrink: 0;
      margin-right: 12px;
    }
    
    .user-details {
      flex: 1;
      
      .user-name-row {
        display: flex;
        align-items: center;
        flex-wrap: wrap;
        gap: 8px;
        
        .user-name {
          font-size: 14px;
          font-weight: 500;
          color: #1d2129;
        }
        
        .self-tag {
          font-size: 12px;
          padding: 1px 6px;
          background: #e8f3ff;
          color: #1e80ff;
          border-radius: 2px;
        }
        
        .comment-time {
          font-size: 12px;
          color: #86909c;
        }
      }
    }
  }
  
  .comment-content {
    font-size: 14px;
    line-height: 1.6;
    color: #1d2129;
    margin-bottom: 12px;
    padding-left: 48px;
  }
  
  .comment-actions {
    display: flex;
    align-items: center;
    gap: 20px;
    padding-left: 48px;
    
    .action-item {
      display: flex;
      align-items: center;
      gap: 4px;
      font-size: 12px;
      color: #86909c;
      cursor: pointer;
      transition: color 0.2s ease;
      user-select: none;
      
      i {
        font-size: 14px;
      }
      
      &:hover {
        color: #1e80ff;
      }
      
      &.active {
        color: #1e80ff;
        
        i {
          color: #1e80ff;
        }
      }
      
      &.delete-btn:hover {
        color: #f53f3f;
      }
      
      .reply-count {
        font-size: 12px;
      }
    }
  }
  
  .reply-input-wrapper {
    margin-top: 12px;
    padding-left: 48px;
    
    .input-wrapper {
      border-radius: 6px;
    }
  }
}

// 子级评论
.child-comments {
  margin-top: 16px;
  margin-left: 48px;
  padding-left: 12px;
  border-left: 2px solid #f0f0f0;
  
  .child-comment-item {
    padding: 12px 0;
    
    &:not(:last-child) {
      border-bottom: 1px solid #f7f8fa;
    }
    
    .child-user-info {
      display: flex;
      align-items: center;
      margin-bottom: 8px;
      
      .child-comment-avatar {
        width: 28px;
        height: 28px;
        margin-right: 8px;
        flex-shrink: 0;
      }
      
      .child-user-details {
        flex: 1;
        
        .child-user-name-row {
          display: flex;
          align-items: center;
          flex-wrap: wrap;
          gap: 8px;
          
          .child-user-name {
            font-size: 13px;
            font-weight: 500;
            color: #1d2129;
          }
          
          .self-tag {
            font-size: 10px;
            padding: 1px 4px;
            background: #e8f3ff;
            color: #1e80ff;
            border-radius: 2px;
          }
          
          .reply-to {
            display: flex;
            align-items: center;
            gap: 4px;
            
            .reply-label {
              font-size: 12px;
              color: #86909c;
              margin: 0 4px;
            }
            
            .replier-avatar {
              width: 16px;
              height: 16px;
              line-height: 16px;
              font-size: 10px;
            }
            
            .replier-name {
              font-size: 12px;
              color: #1e80ff;
              font-weight: 500;
            }
            
            .self-tag.small {
              font-size: 10px;
              padding: 0 4px;
              margin-left: 4px;
            }
          }
          
          .child-comment-time {
            font-size: 11px;
            color: #86909c;
          }
        }
      }
    }
    
    .child-comment-content {
      font-size: 13px;
      line-height: 1.5;
      color: #4e5969;
      margin-bottom: 8px;
      padding-left: 36px;
    }
    
    .child-comment-actions {
      display: flex;
      align-items: center;
      gap: 20px;
      padding-left: 36px;
      
      .action-item {
        font-size: 11px;
        color: #86909c;
        cursor: pointer;
        transition: color 0.2s ease;
        user-select: none;
        display: flex;
        align-items: center;
        gap: 4px;
        
        i {
          font-size: 12px;
        }
        
        &:hover {
          color: #1e80ff;
        }
        
        &.active {
          color: #1e80ff;
          
          i {
            color: #1e80ff;
          }
        }
        
        &.delete-btn:hover {
          color: #f53f3f;
        }
      }
    }
    
    .child-reply-input-wrapper {
      margin-top: 8px;
      padding-left: 36px;
      
      .input-wrapper {
        border-radius: 6px;
      }
    }
  }
}

// 空状态
.empty-state {
  text-align: center;
  padding: 40px 0;
  color: #86909c;
}

// 删除确认对话框
.dialog-footer {
  display: flex;
  justify-content: center;
  gap: 10px;
}

// 响应式设计
@media (max-width: 768px) {
  .evaluations-container {
    padding: 15px;
  }
  
  .comment-header {
    .comment-title {
      font-size: 18px;
    }
  }
  
  .comment-input-wrapper {
    flex-direction: column;
    
    .user-avatar {
      align-self: flex-start;
    }
  }
  
  .comment-main {
    .comment-content,
    .comment-actions,
    .reply-input-wrapper {
      padding-left: 0;
    }
  }
  
  .child-comments {
    margin-left: 20px;
    
    .child-comment-content,
    .child-comment-actions,
    .child-reply-input-wrapper {
      padding-left: 0;
    }
    
    .child-user-info {
      .child-user-details {
        .child-user-name-row {
          .reply-to {
            display: block;
            margin-top: 4px;
          }
        }
      }
    }
  }
}
</style>