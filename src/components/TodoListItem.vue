<template>
  <li :class="todoItemClass" ref="oneItem">
    <div class="heart_wrap" @click="toggleItem">
      <span class="check_heart"></span>
    </div>
    <!-- :ref="'editInput' + index" -->
    <span v-if="!edit.status" class="item stop-dragging">
      <span class="title" @click="editTodo">
        {{ todoItem.title }}
      </span>
      <i class="fa-solid fa-bars" @mousedown="mouseDown"></i>
    </span>

    <span v-else class="item" v-click-outside="onClickOutside">
      <div :style="itemHeight">
        <textarea
          ref="editInput"
          type="text"
          v-model="edit.title"
          @input="editTodoTitle"
          @keyup.enter="editTodoComplete"
          :style="itemHeight"
        />
        <button @click.prevent="removeTodo">
          <i class="fa-solid fa-trash-can"></i>
        </button>
      </div>
    </span>
  </li>
</template>

<script lang="ts">
import { Todo } from "@/App.vue";
import { MyVueRefs } from "@/types";
import Vue, { PropType } from "vue";
import vClickOutside from "v-click-outside";

Vue.use(vClickOutside);

interface Order {
  index: number;
  no: number;
  ydata: number;
}

export default (
  Vue as MyVueRefs<{
    editInput: HTMLTextAreaElement;
    oneItem: HTMLLIElement;
  }>
).extend({
  directives: {
    clickOutside: vClickOutside.directive,
  },
  props: {
    todoItem: {
      type: Object as PropType<Todo>,
    },
    index: Number,
  },
  data() {
    return {
      edit: {
        status: false,
        title: this.todoItem.title,
      },
      isDragging: false,
      order: [] as Order[],
    };
  },
  computed: {
    todoItemClass(): string | null {
      return this.todoItem.done ? "complete" : null;
    },
    itemHeight(): object {
      return { height: this.$refs.oneItem.scrollHeight + "px" };
    },
  },
  methods: {
    onClickOutside(event: PointerEvent) {
      this.editTodoTitle();
      this.editTodoComplete();
    },
    toggleItem() {
      this.$emit("toggle", this.todoItem, this.index);
    },
    editTodo() {
      this.edit.status = true;
      setTimeout(() => {
        const textarea = this.$refs.editInput;
        // textarea.style.height = "auto";
        // textarea.parentElement!.style.height = textarea.scrollHeight + "px";
        // textarea.style.height = textarea.scrollHeight + "px";
        textarea.focus();
      }, 100);
    },
    editTodoTitle() {
      const textarea = this.$refs.editInput;
      textarea.parentElement!.style.height = textarea.scrollHeight + "px";
      textarea.style.height = textarea.scrollHeight + "px";

      this.$emit("edit", this.todoItem, this.index, this.edit.title);
    },
    editTodoComplete() {
      this.edit.status = false;
    },
    removeTodo() {
      this.editTodoComplete();
      this.$emit("remove", this.index);
    },
    mouseDown(e: MouseEvent) {
      const originY = e.clientY;
      const originTop = this.$refs.oneItem.offsetTop;

      this.isDragging = true;
      this.$emit("readyYdata");

      document.onmousemove = e => {
        if (this.isDragging) {
          // const nextItem = this.$refs.oneItem.nextSibling as HTMLLIElement;
          // console.log(nextItem.offsetTop);
          this.$refs.oneItem.style.position = "absolute";
          this.$refs.oneItem.style.boxShadow = "1px 1px 10px rgb(2,2,2)";
          this.$refs.oneItem.style.width = "320px";
          this.$refs.oneItem.style.top = originTop + e.clientY - originY + "px";
        }
      };
      document.onmouseup = e => {
        this.$emit("changeOrder", this.todoItem, e.clientY, this.index);
        this.$refs.oneItem.style.boxShadow = "none";
        this.$refs.oneItem.style.position = "relative";
        this.$refs.oneItem.style.width = "auto";
        this.$refs.oneItem.style.top = "auto";

        this.isDragging = false;
      };
    },
  },
});
</script>

<style></style>
