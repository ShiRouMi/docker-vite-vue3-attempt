<template>
  <section class="todoapp">
      <header class="header">
        <h1>todos</h1>
        <input
          class="new-todo"
          autofocus
          autocomplete="off"
          placeholder="What needs to be done?"
          v-model="state.newTodo"
          @keyup.enter="addTodo"
        />
      </header>
      <section class="main" v-show="state.todos.length" v-cloak>
        <input
          id="toggle-all"
          class="toggle-all"
          type="checkbox"
          v-model="allDone"
        />
        <label for="toggle-all"></label>
        <ul class="todo-list">
          <li
            v-for="todo in filteredTodos"
            class="todo"
            :key="todo.id"
            :class="{ completed: todo.completed, editing: todo == state.editedTodo }"
          >
            <div class="view">
              <input class="toggle" type="checkbox" v-model="todo.completed" />
              <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
              <button class="destroy" @click="removeTodo(todo)"></button>
            </div>
            <input
              class="edit"
              type="text"
              v-model="todo.title"
             
              @blur="doneEdit(todo)"
              @keyup.enter="doneEdit(todo)"
              @keyup.esc="cancelEdit(todo)"
            />
          </li>
        </ul>
      </section>
      <footer class="footer" v-show="state.todos.length" v-cloak>
        <span class="todo-count">
          <strong>{{ remaining }}</strong> {{ rear }} left
        </span>
        <ul class="filters">
          <li>
            <a href="#/all" :class="{ selected: state.visibility == 'all' }">All</a>
          </li>
          <li>
            <a href="#/active" :class="{ selected: state.visibility == 'active' }"
              >Active</a
            >
          </li>
          <li>
            <a
              href="#/completed"
              :class="{ selected: state.visibility == 'completed' }"
              >Completed</a
            >
          </li>
        </ul>
        <button
          class="clear-completed"
          @click="removeCompleted"
          v-show="state.todos.length > remaining"
        >
          Clear completed
        </button>
      </footer>
    </section>
</template>

<script>
// localStorage persistence
var STORAGE_KEY = "todos-vuejs-2.0";
var todoStorage = {
  fetch: function() {
    var todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
    todos.forEach(function(todo, index) {
      todo.id = index;
    });
    todoStorage.uid = todos.length;
    return todos;
  },
  save: function(todos) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
  }
};
// visibility filters
var filters = {
  all: function(todos) {
    // return todos;
    return todos.filter(todo => todo)  // 这里有点奇怪
  },
  active: function(todos) {
    return todos.filter(function(todo) {
      return !todo.completed;
    });
  },
  completed: function(todos) {
    return todos.filter(function(todo) {
      return todo.completed;
    });
  }
};


import { ref, reactive, watch, computed, onMounted } from 'vue'
export default {
  setup() {
    const state = reactive({
      todos: todoStorage.fetch(),
      newTodo: "",
      editedTodo: null,
      visibility: "all"
    })

    const filteredTodos = computed(() => {
      return filters[state.visibility](state.todos);
    })
    const remaining = computed(() => {
      return filters.active(state.todos).length
    })
    const rear = computed(() => remaining === 1 ? "item" : "items")

    const allDone = computed({
      get: () => remaining === 0,
      set: (value) => {
        state.todos.forEach(function(todo) {
          todo.completed = value;
        });
      }
    })

    watch(
      state.todos,
      (todos) => {
        todoStorage.save(todos);
      }
    )

    function addTodo() {
      var value = state.newTodo && state.newTodo.trim();
      if (!value) {
        return;
      }
      state.todos.push({
        id: todoStorage.uid++,
        title: value,
        completed: false
      });
      state.newTodo = "";
    }

    function removeTodo(todo) {
      state.todos.splice(state.todos.indexOf(todo), 1);
    }

    function editTodo(todo) {
      // state.beforeEditCache = todo.title;
      state.editedTodo = todo;
    }

    function doneEdit(todo) {
      if (!state.editedTodo) {
        return;
      }
      state.editedTodo = null;
      todo.title = todo.title.trim();
      if (!todo.title) {
        removeTodo(todo);
      }
    }

    function cancelEdit(todo) {
      state.editedTodo = null;
      // todo.title = this.beforeEditCache;
    }

    function removeCompleted() {
      state.todos = filters.active(state.todos);
    }

    // handle routing
    function onHashChange() {
      debugger
      var visibility = window.location.hash.replace(/#\/?/, "");
      if (filters[visibility]) {
        state.visibility = visibility;
      } else {
        window.location.hash = "";
        state.visibility = "all";
      }
    }
    
    onMounted(() => {
      window.addEventListener("hashchange", onHashChange);
    })

    return {
      state,
      allDone,
      remaining,
      rear,
      addTodo,
      removeTodo,
      editTodo,
      doneEdit,
      cancelEdit,
      removeCompleted,
      filteredTodos
    }
  }
}
</script>
