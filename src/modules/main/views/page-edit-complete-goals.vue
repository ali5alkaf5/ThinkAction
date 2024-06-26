<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import { BaseInput, BaseCheckbox, BaseSelect } from '@/components/index'
import { useUserStore } from '@/stores/user'
import { useRouter, useRoute } from 'vue-router'

const privateTypes = [
  { id: 'public', label: 'Everyone' },
  { id: 'supporter', label: 'Supporter' },
  { id: 'private', label: 'Private' }
]
const userStore = useUserStore()
const router = useRouter()
const route = useRoute()

const id = route.params._id
const categories = ref<any>([])
const goals = ref<any>([])
const currentGoal = ref<any>(null)

onMounted(async () => {
  categories.value = await userStore.getResolutionCategories()
  const data = await userStore.getCurrentGoals()
  goals.value = data.filter(
    (d: any) =>
      d.goal_type !== 'complete' &&
      !data.some((s: any) => s.meta.goal_id === d.id && s.goal_type === 'resolution')
  )

  let goal = userStore.findGoalById(id as string)
  if (goal) {
    form.value = {
      category: { id: goal.category, label: goal.category },
      visibility: privateTypes.find((p) => p.id === (goal as any).visibility) ?? {},
      caption: goal.caption,
      goal: { id: goal.id, label: goal.caption },
      files: goal.photos
    }
    checked.value = !!(goal.meta as any).is_completed
    currentGoal.value = goal
  }
})

const computedGoals = computed(() => {
  return goals.value
    .filter((g: any) => (form.value.category?._id ? g.category === form.value.category?._id : ''))
    .map((a: any) => {
      return {
        id: a._id,
        label: a.caption
      }
    })
})

const checked = ref(false)

const form = ref<any>({
  caption: '',
  category: {},
  goal: {},
  visibility: '',
  files: []
})

const submit = function () {
  let category: any = form.value.category
  let goal: any = form.value.goal
  let visibility: any = form.value.visibility
  let values: any = {}
  if (category._id && goal._id && visibility._id && currentGoal) {
    values.category = category._id
    values.goal_id = goal._id
    values.caption = form.value.caption
    values.visibility = visibility?._id
    values.is_completed = checked.value
    values.files = form.value.files
    userStore.editCompleteGoal(values, (currentGoal.value as any)._id)
    router.push('/')
  }
}
</script>

<template>
  <div v-if="currentGoal" class="main-content-container">
    <p class="text-lg font-semibold">Update Your Complete Goal</p>
    <hr />
    <div>
      <p class="font-semibold text-lg text-[#3D8AF7] text-center mb-8">
        Congratulations! You have achieved your weekly goals, let record them!
      </p>

      <!-- Select Resolution's Category -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Select Category</span>
      <BaseSelect
        :is-error="!(form.category as any)?._id"
        error-message="Choose a category"
        v-model="form.category"
        :list="categories.map((category: string) => ({ id: category, label: category }))"
        class="mb-8"
      ></BaseSelect>

      <!-- Select Goals Achieved -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Goals Achieved</span>
      <BaseSelect
        :is-error="!(form.goal as any)?._id"
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
      <label class="btn btn-primary bg-[#3D8AF7] mb-8 block w-[150px]">
        <input type="file" class="pointer-events-none absolute opacity-0" />
        <div class="flex items-center space-x-2">
          <i class="block i-far-arrow-up-from-bracket"></i>
          <span>Choose File</span>
        </div>
      </label>

      <!-- Checkbox to Complete the Goal -->
      <component :is="BaseCheckbox" v-model="checked" label="Complete Goals" class="mb-8" />

      <!-- share with -->
      <span class="font-semibold text-[#3D8AF7] block mb-2">Share With</span>
      <BaseSelect
        :is-error="!(form.visibility as any)?._id"
        error-message="Choose a visibility"
        v-model="form.visibility"
        :list="privateTypes"
        border="full"
      ></BaseSelect>

      <!-- button -->
      <div class="flex justify-center space-x-2 mt-8">
        <button @click="submit" class="btn btn-primary bg-[#3D8AF7] px-7">SAVE</button>
        <button @click="router.back()" class="btn btn-danger">CANCEL</button>
      </div>
    </div>
  </div>
  <div v-else>Goal not found</div>
</template>
