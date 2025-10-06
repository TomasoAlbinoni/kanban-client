<script setup lang="ts">
const props = defineProps({
  heading: String,
})
const emit = defineEmits<{ (e: 'addNew'): void }>()
</script>

<template lang="pug">
  .column
    h2 {{ props.heading }}
    slot
    button(@click="emit('addNew')") Add Task
</template>
