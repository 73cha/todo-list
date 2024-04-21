<script setup lang="ts">
import { useStorage } from '@vueuse/core'
import { computed } from 'vue'

type todoItem = {
  id: string
  name: string
  done: boolean
}

type todoList = todoItem[]

const todos = useStorage('todos', [] as todoList)

const disableForm = (form: HTMLFormElement) => {
  form.setAttribute('inert', 'true')
}

const resetForm = (form: HTMLFormElement) => {
  form.removeAttribute('inert')
  form.reset()
}

const createTodo = (event: Event) => {
  const form = event.target as HTMLFormElement
  const input = form.querySelector('input[type="text"]') as HTMLInputElement
  const name = input.value

  if (!name) {
    return
  }

  disableForm(form)

  todos.value.push({
    id: crypto.randomUUID().slice(0, 8),
    name,
    done: false,
  })

  setTimeout(() => {
    resetForm(form)
  }, 2000)
}

const leftTodoCount = computed(() => {
  const filtered = todos.value.filter((todo) => todo.done === true)
  return todos.value.length - filtered.length
})

const doneCount = () => {
  const filtered = todos.value.filter((todo) => todo.done === true)
  return filtered.length
}

const toggleStatus = (id: string) => {
  const target = todos.value.find((todo) => todo.id === id)!
  target.done = !target.done
}

const deleteTodo = (id: string) => {
  const filtered = todos.value.filter((todo) => todo.id !== id)
  todos.value = [...filtered]
}

const onDragStart = (event: DragEvent, dragIndex: number) => {
  const dataTransfer = event.dataTransfer as DataTransfer
  dataTransfer.effectAllowed = 'move'
  dataTransfer.dropEffect = 'move'
  dataTransfer.setData('drag-index', String(dragIndex))
}

const onDrop = (event: DragEvent, dropIndex: number) => {
  const dataTransfer = event.dataTransfer as DataTransfer
  const index = Number(dataTransfer.getData('drag-index'))
  const deleted = todos.value.splice(index, 1)
  todos.value.splice(dropIndex, 0, deleted[0])
}
</script>

<template>
  <div class="todo-container">
    <h1 class="app-title">TODOLIST</h1>

    <form class="todo-form" @submit.prevent="createTodo">
      <input class="_input" type="text" placeholder="TODOを入力" required />
      <button class="_button" type="submit">作成</button>
    </form>

    <template v-if="todos.length !== 0">
      <ul class="todo-list" role="list">
        <li
          class="_item"
          v-for="(todo, index) in todos"
          :class="{ 'is-done': todo.done }"
          :key="todo.id"
          :id="todo.id"
          @dragstart="onDragStart($event, index)"
          @drop="onDrop($event, index)"
          @dragover.prevent
          @dragenter.prevent
          draggable="true"
        >
          {{ todo.name }}
          <div>
            <button type="button" @click="toggleStatus(todo.id)">
              {{ !todo.done ? '完了' : '戻す' }}
            </button>
            <button type="button" @click="deleteTodo(todo.id)">削除</button>
          </div>
        </li>
      </ul>
      <p class="todo-message">
        <template v-if="todos.length === doneCount()">
          全てのTODOが完了しています！
        </template>
        <template v-else> {{ leftTodoCount }}つのTODOが未完了です。</template>
      </p>
    </template>
  </div>
</template>
