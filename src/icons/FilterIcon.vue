<template>
  <svg
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-width="2"
    stroke-linecap="round"
    stroke-linejoin="round"
    :class="cls"
  >
    <path d="M3 4h18L14 12v7l-4 1v-8L3 4z" />
  </svg>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  class: { type: [String, Array, Object], default: '' },
})

const cls = computed(() => props.class)
</script>
