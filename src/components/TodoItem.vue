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
    <div class="remove-item" @click="removeTodo(index)">
      &times;
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
      this.$emit("removeTodo", index);
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
      this.$emit("finishedEdit", {
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
  }
};
</script>
