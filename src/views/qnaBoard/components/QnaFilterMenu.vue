<template>
  <div class="flex flex-col gap-4 font-light">
    <span
      class="text-sub-title dark:text-black1 cursor-pointer"
      :class="{ 'font-bold text-primaryRed dark:text-primaryRed': selectedFilter === 'all' }"
      @click="emit('update:modelValue', 'all')"
    >
      전체
    </span>
    <span
      class="text-sub-title dark:text-black1 cursor-pointer"
      :class="{ 'font-bold text-primaryRed dark:text-primaryRed': selectedFilter === 'SOLVED' }"
      @click="emit('update:modelValue', 'SOLVED')"
    >
      해결
    </span>
    <span
      class="text-sub-title dark:text-black1 cursor-pointer"
      :class="{ 'font-bold text-primaryRed dark:text-primaryRed': selectedFilter === 'UNSOLVED' }"
      @click="emit('update:modelValue', 'UNSOLVED')"
    >
      미해결
    </span>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  modelValue: {
    type: String,
    required: true,
  },
})

const emit = defineEmits(['update:modelValue'])

const selectedFilter = computed(() => props.modelValue)
</script>
