<script setup lang="ts">
import type { Item } from '@/types'
import { ref, nextTick, watch } from 'vue'

type Props = {
  item: Item
}

const props = defineProps<Props>()
const emit = defineEmits<{ (e: 'update', value: Item): void }>()

const localItem = ref<Item>(props.item)
watch(
  () => props.item,
  (newItem) => {
    localItem.value = newItem
  },
)
const editingHeading = ref<boolean>(false)
const headingInput = ref<HTMLInputElement | null>(null)

const editingBody = ref<boolean>(false)
const bodyInput = ref<HTMLInputElement | null>(null)

async function headingDoubleClicked(e: MouseEvent) {
  editingHeading.value = true
  await nextTick()
  headingInput.value?.focus()
}

async function bodyDoubleClicked(e: MouseEvent) {
  editingBody.value = true
  await nextTick()
  bodyInput.value?.focus()
}

function editingHeadingEnded() {
  editingHeading.value = false
  emit('update', localItem.value)
}

function editingBodyEnded() {
  editingBody.value = false
  emit('update', localItem.value)
}
</script>

<template lang="pug">
  .item
    .details
      h3(@dblclick="headingDoubleClicked" v-if="!editingHeading")
        | {{ localItem.title }}
      input(type="text" ref="headingInput" v-model="localItem.title" v-else v-on:blur="editingHeadingEnded" @keyup.enter="editingHeadingEnded")
      p(@dblclick="bodyDoubleClicked" v-if="!editingBody")
        | {{ localItem.content }}
      textarea(rows="6" ref="bodyInput" v-model="localItem.content" v-else v-on:blur="editingBodyEnded")
    .controls
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
  margin: 0 1rem 1rem 1rem;
  p {
    white-space: pre;
  }
}

h3 {
  font-size: 1.2rem;
  font-weight: 500;
  margin-bottom: 0.4rem;
  color: $heading_color;
}

textarea {
  width: 100%;
}
</style>
