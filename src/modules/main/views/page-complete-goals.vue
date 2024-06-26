<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import { BaseInput, BaseCheckbox, BaseSelect } from '@/components/index'
import { useUserStore } from '@/stores/user'
import { useRouter } from 'vue-router'
import { usePostStore } from '@/stores/post'

const list = [
  { id: 'everyone', label: 'Everyone' },
  { id: 'supporter', label: 'Supporter' },
  { id: 'private', label: 'Private' }
]

const postStore = usePostStore()
const userStore = useUserStore()
const resolutions = ref<any>([])

const router = useRouter()
const goals = ref<any>([])

onMounted(async () => {
  const user = await userStore.getUserById(userStore.currentUser._id)
  goals.value = await postStore.getPosts()
  resolutions.value = (user.categoryResolution ?? []).filter((u: any) =>
    goals.value.some((g: any) => g.categoryResolutionId === u._id)
  )
})

const computedGoals = computed(() => {
  return goals.value
    .filter((g: any) => g.categoryResolutionId === form.value.category?.id)
    ?.map((g: any) => ({
      id: g._id,
      label: g.caption,
      dueDate: g.dueDate
    }))
})

const checked = ref<any>(false)

const form = ref<any>({
  caption: '',
  category: {},
  goal: {},
  shareWith: {},
  photos: []
})

const submit = async function () {
  let category: any = form.value.category
  let goal: any = form.value.goal
  let shareWith: any = form.value.shareWith
  let values = form.value
  let isAllFilled = category?.id && goal?.id && shareWith?.id && (form.value.category as any)?.id // @ts-ignore-all

  // @ts-ignore

  const formData = new FormData()
  formData.append('caption', values.caption)
  formData.append('weeklyGoalId', (form.value.goal as any)?.id)
  formData.append('categoryResolutionId', (form.value.category as any)?.id)
  formData.append('shareWith', values.shareWith?.id)
  formData.append('dueDate', new Date((form.value.goal as any)?.dueDate).toISOString())
  formData.append('updatedDate', new Date().toISOString())
  formData.append('isComplete', checked.value)
  values.photos?.forEach((photo: any) => {
    formData.append('photo[]', photo)
  })

  if (isAllFilled) {
    await userStore.addCompleteGoal(formData)
    postStore.resetPosts()
    router.push('/')
  }
}
const onImageChange = function (event: any) {
  form.value.photos = [...event.target.files] ?? []
}
</script>

<template>
  <div class="main-content-container">
    <p class="text-lg font-semibold">Create Your Complete Goals</p>
    <hr />
    <div>
      <p class="font-semibold text-lg text-[#3D8AF7] text-center mb-8">
        Congratulations! You have achieved your weekly goals, let record them!
      </p>

      <!-- Select Resolution's Category -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Select Category</span>
      <BaseSelect
        :is-error="!(form.category as any)?.id"
        error-message="Choose a category"
        v-model="form.category"
        :list="resolutions.map((r: any) => ({ id: r?._id, label: r.name }))"
        class="mb-8"
      ></BaseSelect>

      <!-- Select Goals Achieved -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Goals Achieved</span>
      <BaseSelect
        :is-error="!(form.goal as any)?.id"
        error-message="Choose a goal"
        v-model="form.goal"
        :list="computedGoals"
        border="full"
        class="mb-8"
      ></BaseSelect>

      <!-- Caption -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Caption</span>
      <BaseInput
        :error="!form.caption ? 'Enter a caption' : ''"
        v-model="form.caption"
        border="full"
        class="mb-8"
      ></BaseInput>

      <!-- upload photo -->
      <span class="font-semibold text-[#3D8AF7] block mb-2"
        >Share the photo of your vision here</span
      >
      <label class="btn btn-primary bg-[#3D8AF7] mb-8">
        <input
          type="file"
          @change="onImageChange"
          multiple
          class="pointer-events-none absolute opacity-0"
        />
        <div class="flex items-center space-x-2">
          <i class="block i-far-arrow-up-from-bracket"></i>
          <span>Choose File</span>
        </div>
      </label>

      <!-- Checkbox to Complete the Goal -->
      <div class="w-full">
        <component :is="BaseCheckbox" v-model="checked" label="Complete Goals" class="mb-8" />
      </div>

      <!-- share with -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Share With</span>
      <BaseSelect
        :is-error="!(form.shareWith as any)?.id"
        error-message="Choose a shareWith"
        v-model="form.shareWith"
        :list="list"
        border="full"
      ></BaseSelect>

      <!-- button -->
      <div class="flex justify-center space-x-2 mt-8">
        <button @click="submit" class="btn btn-primary bg-[#3D8AF7] px-7">SAVE</button>
        <button class="btn btn-danger">CANCEL</button>
      </div>
    </div>
  </div>
</template>
