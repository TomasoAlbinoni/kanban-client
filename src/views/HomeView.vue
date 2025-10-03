<script setup lang="ts">
import Column from '@/components/KanbanColumn.vue'
import ToDoItem from '@/components/ToDoItem.vue'
import { ref } from 'vue'
import draggable from 'vuedraggable'
import axios from 'axios'
import _ from 'lodash'
import type { Item } from '@/types'

const features = ref<Item[]>([])
const bugs = ref<Item[]>([])
const doing = ref<Item[]>([])
const done = ref<Item[]>([])
const message = ref<string>('')

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
})

function populateLists(items: Item[]) {
  const grouped = _.groupBy(items, 'list')

  features.value = grouped['features'] ? _.sortBy(grouped['features'], 'index') : []
  bugs.value = grouped['bugs'] ? _.sortBy(grouped['bugs'], 'index') : []
  doing.value = grouped['doing'] ? _.sortBy(grouped['doing'], 'index') : []
  done.value = grouped['done'] ? _.sortBy(grouped['done'], 'index') : []
}

async function fetchItems() {
  try {
    const res = await api.get<[Item]>('/')
    populateLists(res.data)
  } catch (err) {
    console.error(err)
  } finally {
    //
  }
}

async function updateItem(item: Item) {
  try {
    const res = await api.put('/update', item)
    message.value = res.data
  } catch (err) {
    console.error(err)
  } finally {
    //
  }
}

function onMoveCallback(column: string, evt: any) {
  if (evt.added || evt.moved) {
    const element: Item = evt.added?.element ?? evt.moved.element
    const newIndex: number = evt.added?.newIndex ?? evt.moved.newIndex
    updateItem({
      ...element,
      list: column,
      index: newIndex,
    } as Item)
  }
}

fetchItems()
</script>

<template lang="pug">
  main
    Column(heading="Features")
      draggable.draggable(v-model="features" item-key="id" group="tasks" @change="(evt) => onMoveCallback('features', evt)")
        template(#item="{ element }")
          ToDoItem(:item="element" @update="updateItem($event)")
    Column(heading="Bugs")
      draggable.draggable(v-model="bugs" item-key="id" group="tasks" @change="(evt) => onMoveCallback('bugs', evt)")
        template(#item="{ element }")
          ToDoItem(:item="element" @update="updateItem($event)")
    Column(heading="Doing")
      draggable.draggable(v-model="doing" item-key="id" group="tasks" @change="(evt) => onMoveCallback('doing', evt)")
        template(#item="{ element }")
          ToDoItem(:item="element" @update="updateItem($event)")
    Column(heading="Done")
      draggable.draggable(v-model="done" item-key="id" group="tasks" @change="(evt) => onMoveCallback('done', evt)")
        template(#item="{ element }")
          ToDoItem(:item="element" @update="updateItem($event)")
  p {{ message.value }}
</template>

<style lang="scss" scoped>
$column_text_color: #222;
$column_bg_color: lightgray;

main {
  display: flex;
  justify-content: space-around;
  align-items: start;
}
.draggable {
  height: 100%;
  min-height: 100px;
}
.column {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  width: 400px;
  background: $column_bg_color;
  margin: 10px;
  border-radius: 5px;
  padding: 10px;
  color: $column_text_color;
}
</style>
