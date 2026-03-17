<template>
  <div class="container">
    <div class="top">
      <el-tooltip class="item" effect="dark" content="返回首页" placement="bottom">
        <span class="last-page" @click="goBack">
          <i class="el-icon-back"></i>
        </span>
      </el-tooltip>
      <span class="text">修改美食帖子</span>
    </div>
    <div class="bottom">
      <div class="edit-card">
        <!-- 标题输入区 -->
        <div class="title-wrap">
          <input 
            v-model="gourmet.title" 
            class="input-title"
            placeholder="请输入标题"
          >
        </div>
        <!-- 富文本编辑器区 -->
        <div class="editor-wrap">
          <Editor 
            height="calc(100vh - 400px)" 
            :receiveContent="gourmet.content" 
            @on-receive="onReceive" 
          />
        </div>
        <!-- 表单操作区 -->
        <div class="form-group">
          <p>*从属菜系</p>
          <el-select 
            size="small" 
            v-model="gourmet.categoryId" 
            placeholder="菜系"
            class="category-select"
          >
            <el-option 
              v-for="item in categories" 
              :key="item.id" 
              :label="item.name" 
              :value="item.id"
            ></el-option>
          </el-select>
        </div>
        <div class="form-group">
          <p>封面</p>
          <el-upload 
            class="avatar-uploader"
            action="http://localhost:21090/api/food-share-sys/v1.0/file/upload" 
            :show-file-list="false"
            :on-success="handleCoverSuccess"
          >
            <img v-if="cover" :src="cover" class="cover-img">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </div>
        <div class="form-group">
          <p>是否公开</p>
          <el-switch 
            v-model="gourmet.isPublish" 
            active-color="#f5a623"
            inactive-color="#e0e0e0"
          ></el-switch>
        </div>
        <div class="btn-wrap">
          <el-button 
            @click="updateGourmet" 
            class="confirm-btn"
            plain
          >确定修改</el-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Editor from "@/components/Editor"
export default {
  components: { Editor },
  name: "CreateGourmet",
  data() {
    return {
      gourmet: {},
      categories: [],
      cover: null,
    }
  },
  created() {
    this.fetchFreshCategories();
    this.loadGourmetId();
  },
  methods: {
    loadGourmetId() {
      const gourmetId = sessionStorage.getItem('gourmetId');
      if (gourmetId) {
        this.fetchGourmetById(Number(gourmetId));
      }
    },
    // 通过ID查找对应的美食做法
    fetchGourmetById(gourmetId) {
      this.$axios.get(`/gourmet/${gourmetId}`).then(res => {
        const { data } = res;
        // 替换?.为传统空值判断，兼容低版本ES
        if (data.code === 200 && data.data && data.data.length) {
          this.gourmet = data.data[0];
          this.cover = this.gourmet.cover;
        }
      }).catch(error => {
        console.log("通过ID查询美食做法异常：", error);
      })
    },
    // 修改美食
    updateGourmet() {
      // 基础校验：标题和内容不能为空
      if (!this.gourmet.title || !this.gourmet.content) {
        this.$notify({
          duration: 1500,
          title: '提示',
          message: '标题和内容不能为空',
          type: 'warning'
        });
        return;
      }
      this.gourmet.cover = this.cover;
      this.$axios.put('/gourmet/update', this.gourmet).then(res => {
        if (res.data.code === 200) {
          this.$notify({
            duration: 1000,
            title: '美食做法修改',
            message: '修改成功',
            type: 'success'
          });
          this.goBack();
        }
      }).catch(error => {
        console.log("修改美食做法异常：", error);
        this.$notify({
          duration: 1500,
          title: '错误',
          message: '修改失败，请重试',
          type: 'error'
        });
      });
    },
    // 封面上传回调函数
    handleCoverSuccess(res, file) {
      const isSuccess = res.code === 200;
      this.$notify({
        duration: 1500,
        title: '封面上传',
        message: isSuccess ? '上传成功' : '上传失败',
        type: isSuccess ? 'success' : 'error'
      });
      if (isSuccess) {
        this.cover = res.data;
      }
    },
    // 查询美食做法的信息
    fetchFreshCategories() {
      this.$axios.post('/category/query', {}).then(res => {
        if (res.data.code === 200) {
          this.categories = res.data.data || [];
          const allCategory = { id: null, name: '全部' };
          this.categories.unshift(allCategory);
        }
      }).catch(error => {
        console.log("查询美食做法异常：", error);
      });
    },
    // 返回上一页
    goBack() {
      this.$router.go(-1);
    },
    onReceive(content) {
      this.gourmet.content = content;
    },
  }
};
</script>

<style scoped lang="scss">
// 全局容器 - 暖黄色基调
.container {
  background-color: #fff9e8;
  min-height: 100vh;

  // 顶部导航栏
  .top {
    line-height: 64px;
    padding-inline: 40px;
    background-color: #f8e4b7;
    border-bottom: 1px solid #e6c883;

    .text {
      font-size: 18px;
      font-weight: 900;
      margin-left: 10px;
      color: #8b5a2b;
    }

    .last-page {
      background-color: #fff;
      padding: 2px 4px;
      border-radius: 12px;
      border: 1px solid #e6c883;
      cursor: pointer;
      color: #8b5a2b;

      &:hover {
        background-color: #fef4d7;
        color: #6d4c41;
      }
    }
  }

  // 底部编辑区
  .bottom {
    margin-top: 12px;
    padding-bottom: 20px;
  }

  // 编辑卡片
  .edit-card {
    width: 900px;
    margin: 0 auto;
    background-color: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(245, 166, 35, 0.1);
    overflow: hidden;
  }

  // 标题输入区
  .title-wrap {
    padding: 20px;
    border-bottom: 1px solid #f5e0b0;

    .input-title {
      width: 100%;
      font-size: 30px;
      border: none;
      outline: none;
      background: transparent;
      color: #8b5a2b;

      &::placeholder {
        color: #d4b483;
        font-weight: normal;
      }
    }
  }

  // 富文本编辑器区
  .editor-wrap {
    border-bottom: 1px solid #f5e0b0;
    padding: 0 20px;
  }

  // 表单组通用样式
  .form-group {
    padding: 15px 20px;
    border-bottom: 1px solid #f5e0b0;

    p {
      margin: 0 0 8px 0;
      color: #8b5a2b;
      font-size: 14px;
    }

    .category-select {
      width: 250px;

      ::v-deep .el-input__inner {
        border-color: #e6c883;
        &:focus {
          border-color: #f5a623;
          box-shadow: 0 0 0 2px rgba(245, 166, 35, 0.2);
        }
      }
    }

    // 封面上传样式
    .avatar-uploader {
      display: inline-block;
      cursor: pointer;

      .cover-img {
        width: 180px;
        height: 130px;
        object-fit: cover;
        border-radius: 4px;
        border: 1px solid #f5e0b0;
      }

      .avatar-uploader-icon {
        font-size: 28px;
        color: #d4b483;
        background-color: #fff9e8;
        width: 180px;
        height: 130px;
        line-height: 130px;
        text-align: center;
        border: 1px dashed #e6c883;
        border-radius: 4px;
      }
    }
  }

  // 按钮区
  .btn-wrap {
    text-align: center;
    padding: 20px 0;

    .confirm-btn {
      background-color: #fff;
      border-color: #f5a623;
      color: #f5a623;
      padding: 8px 30px;
      font-size: 14px;

      &:hover {
        background-color: #f5a623;
        color: #fff;
      }
    }
  }
}
</style>
