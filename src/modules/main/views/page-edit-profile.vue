<script setup lang="ts">
import { ref, computed } from 'vue'
import { BaseInput, BaseSelect } from '@/components/index'
import { useUserStore } from '@/stores/user'
import router from '@/router'
import { getFile } from '@/lib/connection'

const userStore = useUserStore()

const user = computed(() => {
  return userStore.currentUser
})
const uploadedPicture = ref<any>(null)
const showErrors = ref(false)

const list = [
  { id: 1, label: 'Private' },
  { id: 2, label: 'Public' }
]

const languages = ref([
  { id: 1, label: 'English' },
  { id: 2, label: 'Indonesian' }
])

const editProfilePicture = (event: any) => {
  user.value.photo = event.target.files[0]
  uploadedPicture.value = URL.createObjectURL(event.target.files[0])
}

const updateProfile = () => {
  const isAllFilled =
    (preferredLanguage.value as any)?.id &&
    user.value.username &&
    user.value.fullname &&
    user.value.email

  showErrors.value = false
  if (!isAllFilled) {
    showErrors.value = true
  }

  const multipartFormData = new FormData()
  multipartFormData.append('fullname', user.value.fullname)
  multipartFormData.append('username', user.value.username)
  multipartFormData.append('email', user.value.email)
  multipartFormData.append('bio', user.value.bio)
  multipartFormData.append('photo', user.value.photo)
  multipartFormData.append('isPublic', user.value.isPublic)
  userStore.updateProfile(multipartFormData)
  router.push('/profile')
}
const preferredLanguage = ref({ id: 1, label: 'English' })
</script>

<template>
  <div class="grid grid-cols-1 md:p-12 lg:p-4 md:p-[70px] lg:w-[700px] lg:text-md lg:m-auto">
    <div class="grid grid-cols-2">
      <router-link class="flex-grow" :to="{ path: '/profile/edit' }">
        <button class="w-full">
          <h3 class="font-medium border-b-2 text-blue-600 px-3 text-center py-3 text-base">
            Edit Profile
          </h3>
        </button>
      </router-link>
      <router-link class="flex-grow" :to="{ path: '/profile/edit-password' }">
        <button class="w-full">
          <h3 class="text-slate-400 hover:text-blue-600 px-3 text-center py-3 text-base">
            Edit Password
          </h3>
        </button>
      </router-link>
    </div>

    <div class="flex justify-center space-x-5 my-5">
      <img
        :src="uploadedPicture || (user.photo ? getFile(user.photo) : '/profile.png')"
        alt="user photo"
        class="w-32 h-32 object-cover rounded-full"
      />
      <div class="mt-5">
        <p>{{ user.fullname }}</p>

        <!-- CHANGE FOTO PROFILE -->
        <span class="font-semibold text-[#3D8AF7] block mb-2">Change your profile picture</span>
        <label class="btn btn-primary bg-[#3D8AF7] mb-8">
          <input
            type="file"
            @change="editProfilePicture"
            class="pointer-events-none absolute opacity-0"
          />
          <div class="flex items-center space-x-2">
            <i class="block i-far-arrow-up-from-bracket"></i>
            <span>Choose File</span>
          </div>
        </label>
      </div>
    </div>

    <div class="flex flex-col md:px-0 px-3">
      <!-- input - full_name -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Full Name</span>
      <BaseInput
        :error="showErrors && !user.fullname ? 'Enter your full name' : ''"
        v-model="user.fullname"
        class="mb-8"
      ></BaseInput>

      <!-- input - username -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Username</span>
      <BaseInput
        :error="showErrors && !user.username ? 'Enter your username' : ''"
        v-model="user.username"
        class="mb-8"
      ></BaseInput>

      <!-- input - email -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Email</span>
      <BaseInput
        :error="showErrors && !user.email ? 'Enter your email' : ''"
        v-model="user.email"
        class="mb-8"
      ></BaseInput>

      <!-- input - bio -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Bio</span>
      <BaseInput v-model="user.bio" class="mb-8"></BaseInput>

      <!-- share with -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Account Type</span>
      <BaseSelect
        class="mb-8"
        :modelValue="user.isPublic ? list[1] : list[0]"
        @update:modelValue="user.isPublic = $event.id === 2"
        :list="list"
        border="full"
      ></BaseSelect>

      <!-- share with -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Language</span>
      <BaseSelect
        :is-error="showErrors && !(preferredLanguage as any)?.id"
        errorMessage="Choose a main language"
        v-model="preferredLanguage"
        :list="languages"
        border="full"
      ></BaseSelect>

      <div class="flex flex-col justify-center gap-y-2 my-8">
        <button @click="updateProfile" class="btn btn-lg btn-primary bg-[#3D8AF7] px-7">
          Save
        </button>
        <router-link :to="'/profile'" class="btn btn-lg btn-primary bg-[#de2a2a]"
          >Cancel</router-link
        >
      </div>
    </div>
  </div>
</template>
