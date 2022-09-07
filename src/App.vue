<script setup lang="ts">
import { ref, onMounted, watch, computed, Ref } from "vue";

type TodoType = {
  id: number;
  content: string;
  category: string;
  done: boolean;
  editable: boolean;
  createdAt: number;
};
const todos: Ref<TodoType[]> = ref([]);

const name = ref("");
const inputContent = ref("");
const inputCategory: Ref<null | string> = ref(null);

const todosInAscendantOrder = computed(() =>
  [...todos.value].sort((a, b) => a.createdAt - b.createdAt)
);

watch(name, (newName) => {
  localStorage.setItem("name", newName);
});

watch(
  todos,
  (newTodos: TodoType[]) => {
    localStorage.setItem("todos", JSON.stringify(newTodos));
  },
  { deep: true }
);

const addTodo = () => {
  if (!inputCategory.value) return;
  todos.value.push({
    id: Date.now(),
    content: inputContent.value,
    category: inputCategory.value,
    done: false,
    editable: false,
    createdAt: new Date().getTime(),
  });
};

const removeTodo = (todo: TodoType) => {
  todos.value = todos.value.filter((item) => item !== todo);
};

onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  const todosFromDisc = localStorage.getItem("todos");
  if (todosFromDisc) {
    todos.value = JSON.parse(todosFromDisc);
  } else {
    todos.value = [] as TodoType[];
  }
});
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        What's up,
        <input type="text" id="name" placeholder="Name here" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>CREATE A TODO</h3>

      <form id="new-todo-form" @submit.prevent="addTodo">
        <h4>What's on your todo list?</h4>
        <input
          type="text"
          name="content"
          id="content"
          placeholder="e.g. make a video"
          v-model="inputContent"
        />

        <h4>Pick a category</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              id="category1"
              value="business"
              v-model="inputCategory"
            />
            <span class="bubble business"></span>
            <span>Business</span>
          </label>

          <label>
            <input
              type="radio"
              name="category"
              id="category2"
              value="personal"
              v-model="inputCategory"
            />
            <span class="bubble personal"></span>
            <span>Personal</span>
          </label>
        </div>

        <input type="submit" value="Add todo" />
      </form>
    </section>

    <section class="todo-list">
      <h3>TODO LIST</h3>
      <div class="list" id="todo-list">
        <div
          v-for="todo in todosInAscendantOrder"
          :class="`todo-item ${todo.done && 'done'}`"
          :key="todo.id"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span
              :class="`bubble ${
                todo.category === 'business' ? 'business' : 'personal'
              }`"
            ></span>
          </label>

          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>

          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped></style>
