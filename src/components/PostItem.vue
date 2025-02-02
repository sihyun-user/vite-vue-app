<template>
  <section class="post">
    <base-userHeader :user="user" :created-at="createdAt"></base-userHeader>
    <div class="post__content">
      <p class="post__content-text">
        {{ content }}
      </p>
      <div v-if="postImage" class="post__content-img">
        <img :src="postImage">
      </div>
    </div>
    <div class="post__state">
      <div v-if="!isLikePost" class="post__state-like" @click="addPostLike">
        <i class="fa-regular fa-thumbs-up"></i>
        <span>{{ likes.length }}</span>
      </div>
      <div v-else class="post__state-like" @click="canclePostLike">
        <i class="fa-solid fa-thumbs-up"></i>
        <span>{{ likes.length }}</span>
      </div>

      <div class="post__state-msg" @click="switchMsgMode">
        <i class="fa-regular fa-comment"></i>
        <span>{{comments.length }} 則留言</span>
      </div>
    </div>
    <section v-if="isShowComments" class="post__commentWall">
      <div class="post__writeComment">
        <div class="post__writeComment-photo">
          <base-userPhoto :user-photo="userInfo.photo"></base-userPhoto>
        </div>
        <div class="post__writeComment-wrap">
          <input v-model="commentContent" type="text" placeholder="留言...">
          <button @click="addPostComment">留言</button>
        </div>
      </div>
      <div class="post__comments">
        <div v-if="comments.length==0" class="post__no-comment">目前尚無留言，新增一則留言吧！</div>
        <div v-for="comment in comments" :key="comment._id" class="post__commentItem">
          <base-userHeader :user="comment.user" :created-at="comment.createdAt"></base-userHeader>
          <div class="post__commentItem-row">
            <p>{{ comment.comment }}</p>
          </div>
        </div>
      </div>
    </section>
  </section>
</template>

<script>
import { ref, toRefs, computed } from 'vue'
import { useStore } from 'vuex'
import BaseUserHeader from './ui/BaseUserHeader.vue'
import BaseUserPhoto from './ui/BaseUserPhoto.vue'
export default {
  components: {
    BaseUserHeader,
    BaseUserPhoto
  },
  props: {
    postId: {
      type: String
    },
    user: {
      type: Object
    },
    likes: {
      type: Array
    },
    content: {
      type: String
    },
    postImage: {
      type: String
    },
    comments: {
      type: Array
    },
    createdAt: {
      type: String
    },
    showComments: {
      type: Boolean
    }
  },
  emits: ['changeLikes', 'changeComments'],
  setup (props, context) {
    const { postId, likes, showComments } = toRefs(props)
    const store = useStore()
    const isShowComments = ref(false)
    const commentContent = ref('')

    const userId = computed(() => store.getters.userId)
    const userInfo = computed(() => store.getters.userInfo)
    const isLikePost = computed(() => likes.value.includes(userId.value))

    function switchMsgMode () {
      isShowComments.value = !isShowComments.value
    }

    // 新增一則貼文的留言 
    async function addPostComment () {
      await store.dispatch('addPostComment', { 
        postId: postId.value,
        comment: commentContent.value
      })

      commentContent.value = ''

      // 取得貼文的留言
      const result = await store.dispatch('getOnePost', { postId: postId.value })
      if (!result) return
      
      context.emit('changeComments' , {
        postId: result._id,
        comments: result.comments
      })
    }

    // 新增一則貼文的讚
    async function addPostLike () {
      const result = await store.dispatch('addPostLike', { postId: postId.value })
      if (!result) return

      handlePostLike()
    }

    // 取消一則貼文的讚
    async function canclePostLike () {
      const result = await store.dispatch('canclePostLike', { postId: postId.value })
      if (!result) return

      handlePostLike()
    }
    
    // 取得一則貼文的按讚
    async function handlePostLike () {
      // 取得貼文的讚
      const result = await store.dispatch('getOnePost', { postId: postId.value })
      if (!result) return

      context.emit('changeLikes', {
        postId: postId.value,
        newLikes: result.likes
      })
    }

    // 預設是否展開貼文留言
    isShowComments.value = showComments.value

    return {
      isShowComments,
      commentContent,
      userInfo,
      isLikePost,
      switchMsgMode,
      addPostComment,
      addPostLike,
      canclePostLike
    }
  }
}
</script>
