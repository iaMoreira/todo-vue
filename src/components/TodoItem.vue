<template>
  <div class="todo-item">
    <div class="todo-item-left">
      <input type="checkbox" v-model="completed" @change="doneEdit" />
      <div
        v-if="!editing"
        :class="{ completed: completed }"
        @dblclick="editTodo"
        class="todo-item-label"
      >
        {{ title }}
      </div>
      <input
        v-else
        @blur="doneEdit"
        @keyup.enter="doneEdit"
        @keyup.esc="cancelEdit"
        class="todo-item-edit"
        type="text"
        v-model="title"
      />
    </div>
    <div>
      <button @click="pluralize">Plural</button>
      <div class="remove-item" @click="removeTodo(index)">
        &times;
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "todo-item",
  props: {
    todo: {
      type: Object,
      required: true
    },
    index: {
      type: Number,
      required: true
    },
    checkAll: {
      type: Boolean,
      required: true,
    }
  },
  data() {
    return {
      id: this.todo.id,
      title: this.todo.title,
      completed: this.todo.completed,
      editing: this.todo.editing,
      beforeEditCache: this.todo.beforeEditCache
    };
  },
  created() {
    eventBus.$on("pluralize", this.handlePluralize);
  },
  beforeDestroy() {
    eventBus.$off("pluralize", this.handlePluralize);
  },
  watch: {
    checkAll() {      
      this.completed = this.checkAll ? true : this.todo.completed;
    }
  },
  directives: {
    inserted: function(el) {
      el.focus();
    }
  },
  methods: {
    removeTodo(index) {
      eventBus.$emit("removeTodo", index);
    },
    editTodo(){
      this.beforeEditCache = this.title;
      this.editing = true;
    },
    doneEdit() {
      if (this.title.trim() == '') {
        this.title = this.beforeEditCache;
      }
      this.editing = false;
      eventBus.$emit("finishedEdit", {
        "index": this.index,
        "todo": {
          "id": this.id,
          "title": this.title,
          "completed": this.completed,
          "editing": this.editing,
        }
      });
    },
    cancelEdit() {
      this.title = this.beforeEditCache;
      this.editing = false;
    },
    pluralize() {
      eventBus.$emit('pluralize');
    },
    handlePluralize() {
      this.title = this.title + 's';
      eventBus.$emit("finishedEdit", {
        "index": this.index,
        "todo": {
          "id": this.id,
          "title": this.title,
          "completed": this.completed,
          "editing": this.editing,
        }
      });
    }
  }
};
</script>
