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
import { computed, defineComponent, onBeforeMount, onMounted, onBeforeUpdate, onUpdated, onBeforeUnmount, onUnmounted, reactive } from 'vue'
import { Todo } from '@/types/todo'
import TodoInput from '@/components/TodoInput.vue'
import TodoLabel from '@/components/TodoLabel.vue'
import MyCookie from '@/plugins/Cookie'

interface State {
  todoList: Todo[];
}

export default defineComponent({
  components: {
    TodoInput,
    TodoLabel
  },
  setup () {
    // data
    const state = reactive<State>({
      todoList: [] as Todo[]
    })

    // methods
    const loadData = async () => {
      console.log('loadData')

      // データの読み込み処理
      state.todoList = [] as Todo[]
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
        state.todoList.push({
          id: id,
          done: done,
          date: date,
          text: text,
          color: color
        })
      }
    }
    const saveData = () => {
      console.log('saveData')

      // データの書き込み処理
      const cookie: MyCookie = new MyCookie()
      let i = 0
      for (; i < state.todoList.length; i++) {
        const todo = state.todoList[i]
        cookie.setValue('id' + i, todo.id)
        cookie.setBool('done' + i, todo.done)
        cookie.setNumber('date' + i, todo.date.getTime())
        cookie.setValue('text' + i, todo.text)
        cookie.setValue('color' + i, todo.color)
      }
      cookie.setValue('id' + i, '')
    }
    const addTodo = (text: string, color: string) => {
      state.todoList = [...state.todoList, {
        id: (new Date()).getTime().toString(),
        done: false,
        date: new Date(),
        text: text,
        color: color
      }]
      saveData()
    }
    const removeTodo = (id: string) => {
      state.todoList = state.todoList.filter(todo => todo.id !== id)
      saveData()
    }
    const doneTodo = (id: string) => {
      const todo = state.todoList.find(todo => todo.id === id)
      if (todo) {
        todo.done = !todo.done
        saveData()
      }
    }

    // computed
    const sortedTodo = computed(() => {
      return state.todoList.slice().sort((a, b) => {
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
