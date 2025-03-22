<template>
  <div>
    <todo-input
      v-on:add="addTodo"
    />
    <todo-label
      v-for="todo in sortedTodo"
      v-bind:key="todo.id"
      v-bind:todo="todo"
      v-on:done="doneTodo"
      v-on:remove="removeTodo"
    />
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, onBeforeMount, onMounted, onBeforeUpdate, onUpdated, onBeforeUnmount, onUnmounted } from 'vue'
import { Todo } from '@/types/todo'
import TodoInput from '@/components/TodoInput.vue'
import TodoLabel from '@/components/TodoLabel.vue'
import MyCookie from '@/plugins/Cookie'
import store from '@/store'

export default defineComponent({
  components: {
    TodoInput,
    TodoLabel
  },
  setup () {
    // methods
    const loadData = async () => {
      console.log('loadData')

      // データの読み込み処理
      store.dispatch('initialize')
      const cookie: MyCookie = new MyCookie()
      for (let i = 0; ; i++) {
        const id = cookie.getValue('id' + i, '')
        if (id.length === 0) {
          break
        }
        const done = cookie.getBool('done' + i, false)
        const date = new Date(cookie.getNumber('date' + i, 0))
        const text = cookie.getValue('text' + i, '')
        const color = cookie.getValue('color' + i, '')
        store.dispatch('push', {
          todo: {
            id: id,
            done: done,
            date: date,
            text: text,
            color: color
          }
        })
      }
    }
    const saveData = () => {
      console.log('saveData')

      // データの書き込み処理
      const todoList = store.getters.todoList
      const cookie: MyCookie = new MyCookie()
      let i = 0
      for (; i < todoList.length; i++) {
        const todo = todoList[i]
        cookie.setValue('id' + i, todo.id)
        cookie.setBool('done' + i, todo.done)
        cookie.setNumber('date' + i, todo.date.getTime())
        cookie.setValue('text' + i, todo.text)
        cookie.setValue('color' + i, todo.color)
      }
      cookie.setValue('id' + i, '')
    }
    const addTodo = (text: string, color: string) => {
      store.dispatch('add', {
        todo: {
          id: (new Date()).getTime().toString(),
          done: false,
          date: new Date(),
          text: text,
          color: color
        }
      })
      saveData()
    }
    const removeTodo = (id: string) => {
      store.dispatch('remove', { id: id })
      saveData()
    }
    const doneTodo = (id: string) => {
      store.dispatch('done', { id: id })
      saveData()
    }

    // computed
    const sortedTodo = computed(() => {
      const todoList = store.getters.todoList.slice()
      return todoList.sort((a: Todo, b: Todo) => {
        return b.date.getTime() - a.date.getTime()
      })
    })

    onBeforeMount(() => {
      console.log('MyTodo onBeforeMount')
    })
    onMounted(async () => {
      console.log('MyTodo onMounted')

      // データの読み込み
      await loadData()
    })
    onBeforeUpdate(() => {
      console.log('MyTodo onBeforeUpdate')
    })
    onUpdated(() => {
      console.log('MyTodo onUpdated')
    })
    onBeforeUnmount(() => {
      console.log('MyTodo onBeforeUnmount')
    })
    onUnmounted(() => {
      console.log('MyTodo onUnmounted')
    })

    // template内で使用するプロパティ
    return {
      addTodo,
      removeTodo,
      doneTodo,
      sortedTodo
    }
  }
})
</script>
