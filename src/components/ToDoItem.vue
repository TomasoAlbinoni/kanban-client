<script setup lang="ts">
import type { Item } from '@/types'
import { ref, nextTick, watch } from 'vue'

type Props = {
  item: Item
}

const props = defineProps<Props>()
// TODO const emit = defineEmits<{ (e: 'update:title', value: string): void }>()

const localTitle = ref(props.item.title)
watch(
  () => props.item,
  (newItem) => {
    localTitle.value = newItem.title
  },
)
const editingHeading = ref<boolean>(false)
const headingInput = ref<HTMLInputElement | null>(null)

async function headingDoubleClicked(e: MouseEvent) {
  editingHeading.value = true
  await nextTick()
  headingInput.value?.focus()
}

function editingHeadingEnded() {
  editingHeading.value = false
}
</script>

<template lang="pug">
  .item
    .details
      h3(@dblclick="headingDoubleClicked" v-if="!editingHeading")
        | {{ item.title }}
      input(type="text" ref="headingInput" v-model="localTitle" v-else v-on:blur="editingHeadingEnded" @keyup.enter="editingHeadingEnded")
      p {{ item.content }}
</template>

<style lang="scss" scoped>
$background_color: #eee;
$border_color: #bbb;
$heading_color: #333;
.item {
  margin-top: 10px;
  display: flex;
  position: relative;
  background-color: $background_color;
  border-radius: 5px;
  border: 1px solid $border_color;
}

.details {
  flex: 1;
  margin-left: 1rem;
}

h3 {
  font-size: 1.2rem;
  font-weight: 500;
  margin-bottom: 0.4rem;
  color: $heading_color;
}
</style>
