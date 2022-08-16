<template>
  <div class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        autofocus
        autocomplete="off"
        placeholder="What needs to be done?"
        v-model="newTodo"
        @keyup.enter="handleAddTodo"
      />
    </header>
    <section class="main">
      <input
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
        v-model="allCompleted"
      />
      <label
        for="toggle-all"
        v-show="data.todos.length"
      >
        Mark all as complete
      </label>
      <ul class="todo-list">
        <li
          :class="['todo',
          {completed: todo.completed},
          {editing: editTodo.id === todo.id}]"
          v-for="todo in showTodos"
          :key="todo.id"
        >
          <div class="view">
            <input
              class="toggle"
              type="checkbox"
              v-model="todo.completed"
            />
            <label
              @dblclick="handleEdit(todo)"
            >
              {{todo.title}}
            </label>
            <button
              class="destroy"
              @click="removeTodo(todo)"
            ></button>
          </div>
          <input
            class="edit"
            type="text"
            v-model.trim="todo.title"
            v-todo-focus="editTodo.id === todo.id"
            @blur="handleEditDone(todo)"
            @keyup.enter="handleEditDone(todo)"
            @keyup.esc="handleEditCancel(todo)"
          />
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="data.todos.length">
      <span class="todo-count"> <strong>{{count}}</strong>  items left </span>
      <ul class="filters">
        <li>
          <a
            href="#/all"
            :class="{selected: filter === 'all'}"
            @click="handleFilter('all')"
          >
            All
          </a>
        </li>
        <li>
          <a
            href="#/active"
            :class="{selected: filter === 'active'}"
            @click="handleFilter('active')"
          >
            Active
          </a>
        </li>
        <li>
          <a
            href="#/completed"
            :class="{selected: filter === 'completed'}"
            @click="handleFilter('completed')"
          >
            Completed
          </a>
        </li>
      </ul>
      <button
        class="clear-completed"
        v-show="hasCompleted"
        @click="removecompleted"
      >
        Clear completed
      </button>
    </footer>
  </div>
</template>

<script setup>
import "@/assets/index.css";
import { ref, reactive, computed } from "vue";

const newTodo = ref("");
const filter = ref("all");
const id = ref(1);
const beforeEditCache = ref("");
const editTodo = reactive({
  id: null,
  title: null,
});
const data = reactive({
  todos: [],
});

const allCompleted = computed({
  get() {
    return data.todos.every((data) => data.completed);
  },
  set(value) {
    data.todos.forEach((data) => data.completed = value);
  },
});
const showTodos = computed(() => {
  return data.todos.filter((data) => {
    if (filter.value === "all") {
      return true;
    } else if (filter.value === "active") {
      return !data.completed;
    } else if (filter.value === "completed") {
      return data.completed;
    }
    return false;
  })
});
const count = computed(() => {
  return data.todos.filter((data) => !data.completed).length;
});
const hasCompleted = computed(() => {
  return data.todos.some((data) => data.completed);
});

const handleAddTodo = () => {
  const value = newTodo.value.trim();
  if (!value) {
    return;
  }
  data.todos.push({ id: id.value++, title: value, completed: false });
  newTodo.value = "";
};
const handleEdit = (todo) => {
  editTodo.id = todo.id;
  editTodo.title = todo.title;
  beforeEditCache.value = todo.title;
};
const removeTodo = (todo) => {
  const index = data.todos.indexOf(todo);
  data.todos.splice(index, 1);
};
const handleEditDone = (todo) => {
  editTodo.id = null;
  editTodo.title = null;
  beforeEditCache.value = null;
  if (!todo.title) {
    removeTodo(todo);
  }
};
const handleEditCancel = (todo) => {
  editTodo.id = null;
  editTodo.title = null;
  todo.title = beforeEditCache.value;
  beforeEditCache.value = null;
};
const handleFilter = (value) => {
  filter.value = value;
};
const removecompleted = () => {
  data.todos = data.todos.filter((data) => !data.completed);
}

const vTodoFocus = {
  updated(el, binding) {
    if (binding.value) {
      el.focus();
    }
  }
};
</script>