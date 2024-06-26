<script lang="ts"></script>
<script setup lang="ts">
import type {
  ThinkActionUser,
  ThinkActionCategory,
  ThinkActionGoal
} from '../../../types/think-action'
import { computed, ref } from 'vue'
import { RouterLink } from 'vue-router'
import BaseProfileHeader from '../profile-header.vue'
import BaseResolutionCategory from '../profile-resolution-categories.vue'
import BaseUserPost from '../user-post.vue'
import { useUserStore } from '@/stores/user'

type Props = {
  is_current_user?: boolean
  user: ThinkActionUser | null
  posts: ThinkActionGoal[]
  categories?: ThinkActionCategory[]
}

const props = withDefaults(defineProps<Props>(), {
  is_current_user: false,
  user: null
})

const userStore = useUserStore()

const emit = defineEmits(['update'])

const selectedCategory = ref('')

const selectCategory = function (id: string) {
  selectedCategory.value = id
}

const userPosts = computed(() => {
  if (selectedCategory.value === '') {
    return props.posts
  }
  return props.posts.filter((p: ThinkActionGoal) => {
    return p.name === selectedCategory.value
  })
})

const computedCategories = computed(() => {
  return props.posts
    .reduce((p, u) => (p.includes(u.name as string) ? p : [...p, u.name as string]), [''])
    .slice(1)
})
// TODO: Add async functions
const supportUser = async function () {
  const data = await userStore.toggleSupport(
    (props.user as any)._id,
    (props.user as any).isSupporting
  )
  emit('update', {
    ...props.user,
    ...data
  })
}
// onMounted(() => {
//   isSupporting.value = userStore.isSupporting((props.user as any)._id)
// })
const goalsPerformance = computed(() => {
  const length = props.user?.categoryResolution?.length ?? 1
  const matched = props.user?.categoryResolution?.filter((c: any) => c.isComplete)?.length
  return Math.round((matched / length) * 10000) / 100
})
</script>

<template>
  <div v-if="props.user" class="main-content-container">
    <!-- PROFILE HEADER -->
    <BaseProfileHeader
      :id="props.user._id"
      :full_name="props.user.fullname!"
      :username="props.user.username"
      :avatar="props.user.photo"
      :bio="props.user.bio!"
      :goals-performance="goalsPerformance"
      :is_private="!props.user.isPublic!"
      :is-supporting="props.user.isSupporting"
      @support="supportUser"
      :goals_performance="props.user.goals_performance!"
      :supporting-count="props.user.supportingCount!"
      :supporter-count="props.user.supporterCount!"
      :is_user="props.is_current_user"
    ></BaseProfileHeader>

    <div v-if="!props.is_current_user">
      <!-- SUPPORTING BUTTON -->
      <button
        @click="supportUser"
        v-if="!props.user.isSupporting"
        class="btn bg-[#3D8AF7] w-full font-semibold text-white md:hidden"
      >
        Support
      </button>
      <template v-if="props.user.isSupporting">
        <button
          @click="supportUser"
          class="btn bg-slate-300 text-sky-500 w-full font-semibold text-white md:hidden"
        >
          Supporting
        </button>
        <!-- <button class="  btn bg-slate-400 w-full font-semibold text-white md:hidden">Unsupport</button> -->
      </template>
    </div>
    <router-link v-else :to="{ path: '/profile/edit' }">
      <button class="btn bg-[#3D8AF7] w-full font-semibold text-white mt-5">Edit Profile</button>
    </router-link>

    <!-- RESOLUTIONS -->
    <div>
      <hr class="border" />
      <BaseResolutionCategory
        :selected_category_id="selectedCategory"
        :resolution_categories="computedCategories"
        @select="selectCategory"
      ></BaseResolutionCategory>
      <hr class="border" />
    </div>

    <div class="space-y-2">
      <!-- USER POSTS -->
      <div v-for="post in userPosts" :key="post._id">
        <BaseUserPost
          :goal_type="post.name"
          :key="post._id"
          :id="post._id"
          :user_id="user?._id!"
          :user="user!"
          :category="post.name!"
          :caption="post.resolution"
          :photos="post.photo"
          :is_liked="post.is_liked_by_user"
          :cheers_count="post.cheers_count"
          :comments_count="post.comments_count"
          :date_time="post.date_time"
          :created_at="post.createdDate"
        ></BaseUserPost>
      </div>
    </div>
  </div>
</template>
