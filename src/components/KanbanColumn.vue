<script setup lang="ts">
const props = defineProps({
  heading: String,
})
</script>

<template lang="pug">
  .column
    h2 {{ props.heading }}
    slot
</template>
