<template>
  <div ref="loadingContainer" class="collection-container">
    <div v-if="followers.length === 0 && !loading">
      <el-empty description="现在还没有点赞消息..." />
    </div>
    <div v-else>
    <ul class="agree-container">
      <li class="agree-item" v-for="follower in followers" :key="follower.createTime">
        <a class="user-avatar">
          <img class="avatar-item" :src="follower.avatar" />
        </a>
        <div class="main">
          <div class="info">
            <div class="user-info">
              <a>{{ follower.username }}</a>
            </div>
            <div class="interaction-hint"><span>开始关注您了&nbsp;</span><span>{{ formatDate(follower.createTime) }}</span></div>
          </div>
          <div class="extra">
            <button
              class="reds-button-new follow-button large"
              :class="{'primary': !follower.back, 'mutual': follower.back}"
              @click="toggleFollow(follower)"
            >
              <span class="reds-button-new-box"><span class="reds-button-new-text">{{ follower.back ? '互相关注' : '回关' }}</span></span>
            </button>
          </div>
        </div>
      </li>
    </ul>
  </div>
</div>
</template>
<script lang="ts" setup>
import { ref, onMounted } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { getFocusInfo,doFocus,unFollow } from '@/apis/main';
import { ElLoading } from 'element-plus';
import 'element-plus/theme-chalk/el-loading.css';

const route = useRoute();
const loading = ref(false);
const loadingContainer = ref(null);
let loadingInstance: any = null;


interface Follower {
  username: string;
  avatar: string;
  createTime: string;
  back: boolean;
}

const followers = ref<Follower[]>([]);

const fetchFollowers = async () => {
  const user_id = route.params.id;
  try {
    loading.value = true;
    loadingInstance = ElLoading.service({
      target: loadingContainer.value,
      lock: true,
      text: '加载中...',
      background: 'rgba(0, 0, 0, 0.7)',
    });
    const post = await getFocusInfo({ user_id });
    followers.value = post.info;
  } catch (error) {
    console.error('Error fetching messages:', error);
  } finally {
    loading.value = false;
    if (loadingInstance) {
      loadingInstance.close();
    }
  }
};

const toggleFollow = async (follower: Follower) => {
  try {
    if (follower.back) {
      await axios.post(`/api/unfollow`, { userId: follower.id });
      follower.back = false;
    } else {
      await axios.post(`/api/focusOn`, { userId: follower.id });
      follower.back = true;
    }
  } catch (error) {
    console.error('Error toggling follow status:', error);
  }
};

const formatDate = (dateString: string) => {
  const date = new Date(dateString);
  return date.toLocaleDateString();
};

onMounted(() => {
  fetchFollowers();
});
</script>
<style lang="less" scoped>
textarea {
  overflow: auto;
}
.agree-container {
  width: 40rem;
  .agree-item {
    display: flex;
    flex-direction: row;
    padding-top: 24px;

    .user-avatar {
      margin-right: 24px;
      flex-shrink: 0;

      .avatar-item {
        width: 48px;
        height: 48px;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        border-radius: 100%;
        border: 1px solid rgba(0, 0, 0, 0.08);
        object-fit: cover;
      }
    }

    .main {
      flex-grow: 1;
      flex-shrink: 1;
      display: flex;
      flex-direction: row;
      padding-bottom: 12px;
      border-bottom: 1px solid rgba(0, 0, 0, 0.08);

      .info {
        flex-grow: 1;
        flex-shrink: 1;

        .user-info {
          display: flex;
          flex-direction: row;
          align-items: center;
          justify-content: space-between;
          margin-bottom: 4px;
          a {
            color: #333;
            font-size: 16px;
            font-weight: 600;
          }
        }
        .interaction-hint {
          font-size: 14px;
          color: rgba(51, 51, 51, 0.6);
          margin-bottom: 8px;
        }
      }

      .extra {
        min-width: 48px;
        flex-shrink: 0;
        margin-left: 24px;
        .follow-button {
          width: 96px;
          &.primary {
            background-color: #2f779d;
            color: #fff;
          }
          &.mutual {
            background-color: #ccc;
            color: #333;
          }
        }
        .reds-button-new.large {
          font-size: 16px;
          font-weight: 600;
          line-height: 16px;
          padding: 0 24px;
          height: 40px;
        }
        .reds-button-new {
          position: relative;
          cursor: pointer;
          -webkit-user-select: none;
          user-select: none;
          white-space: nowrap;
          outline: none;
          background: none;
          border: none;
          vertical-align: middle;
          text-align: center;
          display: inline-block;
          padding: 0;
          border-radius: 100px;
          font-weight: 500;
        }
      }
    }
  }
}
</style>