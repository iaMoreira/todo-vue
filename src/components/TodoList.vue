<template>
  <div>
    <input
      type="text"
      class="todo-input"
      placeholder="What needs to be done"
      v-model="newTodo"
      @keyup.enter="addTodo"
    />
    <transition-group
      name="fade"
      enter-active-class="animated fadeInUp"
      leave-active-class="animated fadeOutDown"
    >
      <todo-item
        v-for="(todo, index) in todosFiltered"
        :key="todo.id"
        :todo="todo"
        :index="index"
        :checkAll="!anyRemaining"
      >
      </todo-item>
    </transition-group>

    <div class="extra-container">
      <todo-check-all :anyRemaining="anyRemaining"></todo-check-all>
      <todo-items-remaining :remaining="remaining"></todo-items-remaining>
    </div>

    <div class="extra-container">
    <todo-filtered :filter="filter"></todo-filtered>

      <div>
        <transition name="fade">
          <todo-clear-completed :showClearCompletedButton="showClearCompletedButton"></todo-clear-completed>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
import TodoItem from "./TodoItem";
import TodoItemsRemaining from "./TodoItemsRemaining";
import TodoCheckAll from "./TodoCheckAll";
import TodoFiltered from "./TodoFiltered";
import TodoClearCompleted from "./TodoClearCompleted";
export default {
  name: "todo-list",
  components: {
    TodoItem,
    TodoItemsRemaining,
    TodoCheckAll,
    TodoFiltered,
    TodoClearCompleted
  },
  data() {
    return {
      newTodo: "",
      idForTodo: 3,
      beforeEditCache: "",
      filter: "all",
      todos: [
        {
          id: 1,
          title: "Finish Vue Screencast",
          completed: false,
          editing: false
        },
        {
          id: 2,
          title: "take over world",
          completed: false,
          editing: false
        }
      ]
    };
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim().length == 0) {
        return;
      }
      this.todos.push({
        id: this.idForTodo,
        title: this.newTodo,
        completed: false,
        editing: false
      });
      this.newTodo = "";
      this.idForTodo++;
    },
    removeTodo(index) {
      this.todos.splice(index, 1);
    },
    checkAllTodos() {
      this.todos.forEach((todo) => todo.completed = event.target.checked);
    },
    clearCompleted() {
      this.todos = this.todos.filter(todo => !todo.completed);
    },
    finishedEdit(data) {
      const {index, todo} = data;
      this.todos.splice(index, 1, todo);
    }
  },
  created() {
    eventBus.$on("removeTodo", (index) => this.removeTodo(index));
    eventBus.$on("finishedEdit", (index) => this.finishedEdit(index));
    eventBus.$on("checkAllChanged", (checked) => this.checkAllTodos(checked));
    eventBus.$on("filterChanged", (filter) => this.filter = filter);
    eventBus.$on("clearCompletedTodos", () => this.clearCompleted());
  },
  beforeDestroy() {
    eventBus.$off("removeTodo", (index) => this.removeTodo(index));
    eventBus.$off("finishedEdit", (index) => this.finishedEdit(index));
    eventBus.$off("checkAllChanged", (checked) => this.checkAllTodos(checked));
    eventBus.$off("filterChanged", (filter) => this.filter = filter);
    eventBus.$off("clearCompletedTodos", () => this.clearCompleted());
    
  },
  computed: {
    remaining() {
      return this.todos.filter(todo => !todo.completed).length;
    },
    anyRemaining() {
      return this.remaining != 0;
    },
    todosFiltered() {
      if (this.filter == "all") {
        return this.todos;
      } else if (this.filter == "active") {
        return this.todos.filter(todo => !todo.completed);
      } else if (this.filter == "completed") {
        return this.todos.filter(todo => todo.completed);
      }
      return this.todos;
    },
    showClearCompletedButton() {
      return this.todos.filter(todo => todo.completed).length > 0;
    }
  },
  directives: {
    inserted: function(el) {
      el.focus();
    }
  }
};
</script>

<style>
@import "../assets/scss/main.scss";
</style>
