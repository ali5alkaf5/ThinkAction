<script setup lang="ts">
import { getFile } from '@/lib/connection'

export interface Props {
  id: string
  fullname: string
  username: string
  avatar: string
  // supporter: Array<User>
}

const props = withDefaults(defineProps<Props>(), {
  avatar: '/public/profile.png'
})
</script>

<template>
  <div class="flex mb-5">
    <img
      :src="props.avatar ? getFile(props.avatar) : '/public/profile.png'"
      class="w-[60px] h-[60px] object-cover inline-block mr-2 rounded-full"
      alt="user profile"
    />
    <div>
      <p class="font-semibold">{{ props.username }}</p>
      <p class="">{{ props.fullname }}</p>
      <div class="flex">
        <!-- <p class="">Supported by {{ props.supporter[0].username + ' ' + '+' }}</p> -->
        <!-- <p>{{ props.supporter.length - 1 }} more</p> -->
      </div>
    </div>
  </div>
</template>
