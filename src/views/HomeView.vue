<script setup lang="ts">
import { ref } from 'vue'
import _ from 'lodash'
import type { AxiosError } from 'axios'
import api from '@/lib/api'
import type { Item } from '@/types'
import Column from '@/components/KanbanColumn.vue'
import ToDoItem from '@/components/ToDoItem.vue'
import draggable from 'vuedraggable'

const features = ref<Item[]>([])
const bugs = ref<Item[]>([])
const doing = ref<Item[]>([])
const done = ref<Item[]>([])
const message = ref<string>('')

function populateLists(items: Item[]) {
  const grouped = _.groupBy(items, 'list')

  features.value = grouped['features'] ? _.sortBy(grouped['features'], 'index') : []
  bugs.value = grouped['bugs'] ? _.sortBy(grouped['bugs'], 'index') : []
  doing.value = grouped['doing'] ? _.sortBy(grouped['doing'], 'index') : []
  done.value = grouped['done'] ? _.sortBy(grouped['done'], 'index') : []
}

async function fetchItems() {
  try {
    const res = await api.get<[Item]>('/tasks/')
    populateLists(res.data)
  } catch (err) {
    console.error(err)
    const error = err as AxiosError
    if (error.status === 401 || error.status === 403) {
      window.location.href = '/login'
    }
  } finally {
    //
  }
}

async function updateItem(item: Item, column: string, index: number) {
  try {
    const res = await api.put('/tasks/update', {
      ...item,
      list: column,
      index: index,
    } as Item)
    message.value = res.data
  } catch (err) {
    console.error(err)
    const error = err as AxiosError
    if (error.status === 401 || error.status === 403) {
      window.location.href = '/login'
    }
  } finally {
    //
  }
}

async function deleteItem(item: Item) {
  try {
    item.deleting = true
    const res = await api.delete('/tasks/delete/' + item.id)
    message.value = res.data
    let array: Item[] = getArrayRef(item.list)
    const index = array.indexOf(item)
    if (index !== -1) {
      array.splice(index, 1)
    }
  } catch (err) {
    item.deleting = false
    console.error(err)
    const error = err as AxiosError
    if (error.status === 401 || error.status === 403) {
      window.location.href = '/login'
    }
  } finally {
    //
  }
}

function onMoveCallback(column: string, evt: any) {
  if (evt.added || evt.moved) {
    const element: Item = evt.added?.element ?? evt.moved.element
    const newIndex: number = evt.added?.newIndex ?? evt.moved.newIndex
    updateItem(element, column, newIndex)
  }
}

function getArrayRef(column: string): Item[] {
  switch (column) {
    case 'features':
      return features.value
    case 'bugs':
      return bugs.value
    case 'doing':
      return doing.value
    case 'done':
      return done.value
  }
  return []
}

async function onAddNewClicked(column: string) {
  const newItem: Item = {
    id: -1,
    list: column,
    title: 'New Task',
    content: '',
    index: -1,
  }
  let array: Item[] = getArrayRef(column)
  array.push(newItem)
  try {
    const res = await api.post('/tasks/create', newItem)
    newItem.id = res.data.id
  } catch (err) {
    const index = array.indexOf(newItem)
    if (index !== -1) {
      array.splice(index, 1)
    }
    console.error(err)
    const error = err as AxiosError
    if (error.status === 401 || error.status === 403) {
      window.location.href = '/login'
    } else {
      console.log(error)
    }
  } finally {
    //
  }
}

fetchItems()
</script>

<template lang="pug">
  main
    Column(heading="Features" @addNew="onAddNewClicked('features')")
      draggable.draggable(v-model="features" item-key="id" group="tasks" @change="(evt) => onMoveCallback('features', evt)")
        template(#item="{ element, index }")
          ToDoItem(:item="element" @update="updateItem($event, 'features', index)" @delete="deleteItem($event)")
    Column(heading="Bugs" @addNew="onAddNewClicked('bugs')")
      draggable.draggable(v-model="bugs" item-key="id" group="tasks" @change="(evt) => onMoveCallback('bugs', evt)")
        template(#item="{ element, index }")
          ToDoItem(:item="element" @update="updateItem($event, 'bugs', index)" @delete="deleteItem($event)")
    Column(heading="Doing" @addNew="onAddNewClicked('doing')")
      draggable.draggable(v-model="doing" item-key="id" group="tasks" @change="(evt) => onMoveCallback('doing', evt)")
        template(#item="{ element, index }")
          ToDoItem(:item="element" @update="updateItem($event, 'doing', index)" @delete="deleteItem($event)")
    Column(heading="Done" @addNew="onAddNewClicked('done')")
      draggable.draggable(v-model="done" item-key="id" group="tasks" @change="(evt) => onMoveCallback('done', evt)")
        template(#item="{ element, index }")
          ToDoItem(:item="element" @update="updateItem($event, 'done', index)" @delete="deleteItem($event)")
  p {{ message.value }}
</template>

<style lang="scss" scoped>
$column_text_color: #222;
$column_bg_color: lightgray;

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
