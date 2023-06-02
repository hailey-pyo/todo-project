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
          rows="1"
          ref="editInput"
          type="text"
          :value="validateTitle"
          @input="editTodoTitle"
          @keyup.enter="editTodoComplete"
          :style="itemHeight"
          autofocus
        />
        <button @click.prevent="removeTodo">
          <i class="fa-solid fa-trash-can"></i>
        </button>
      </div>
    </span>
  </li>
</template>

<script lang="ts">
import { ChangeStatus, Todo } from "@/App.vue";
import { MyVueRefs, VueEvent } from "@/types";
import Vue, { PropType } from "vue";
import vClickOutside from "v-click-outside";
import { throttle } from "lodash";

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
    changeStatus: Object as PropType<ChangeStatus>,
  },
  data() {
    return {
      edit: {
        status: false,
        title: this.todoItem.title,
      },
      isDragging: false,
      order: [] as Order[],
      originY: 0,
    };
  },
  computed: {
    todoItemClass(): string | null {
      return this.todoItem.done ? "complete" : null;
    },
    itemHeight(): object {
      return { height: this.$refs.oneItem.scrollHeight + "px" };
    },
    validateTitle(): string {
      return this.todoItem.title.replace(/\s{2,}/gi, " ");
    },
  },
  methods: {
    onClickOutside() {
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
        textarea.parentElement!.style.height = textarea.scrollHeight + "px";
        textarea.style.height = textarea.scrollHeight + "px";
        textarea.focus();
      }, 100);
    },
    editTodoTitle(event: VueEvent.Input<HTMLInputElement>) {
      if (event.inputType == "insertLineBreak") {
        return;
      }
      this.edit.title = event.target.value;
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
      this.originY = e.clientY;
      console.log("============originY셋팅==========" + this.originY);

      const originTop = this.$refs.oneItem.offsetTop;

      this.isDragging = true;
      this.$emit("readyYdata");

      // const throttle = (callback, delay) => {
      //   let timer;
      //   return event => {
      //     // 타이머가 호출되면, 함수를 실행하고 타이머 제거
      //     if (timer) return;
      //     timer = setTimeout(() => {
      //       callback(event);
      //       timer = null;
      //     }, delay);
      //   };
      // };
      let prevOriginY = this.originY;

      document.addEventListener(
        "mousemove",
        throttle((e: MouseEvent) => {
          if (this.isDragging) {
            if (this.changeStatus.status) {
              prevOriginY = this.originY;
              console.log("e.clientY: " + e.clientY);

              this.$refs.oneItem.style.transform = "translateY(0px)";

              this.originY = e.clientY;
              console.log("순서변경 완료 originY변경! " + this.originY);
              this.$emit("resetChangeStatus", false);
            } else {
              // console.log("originY: " + this.originY);

              // const nextItem = this.$refs.oneItem.nextSibling as HTMLLIElement;
              // console.log(nextItem.offsetTop);

              // this.$refs.oneItem.style.position = "absolute";
              this.$refs.oneItem.style.boxShadow = "1px 1px 10px rgb(2,2,2)";
              // this.$refs.oneItem.style.width = "320px";
              // this.$refs.oneItem.style.top = originTop + e.clientY - originY + "px";

              //322로 돌아와야하는뎅
              //331 296 만약 바뀌면 transform 0 & 현재 originY = e.clientY
              const dy =
                e.clientY <= this.changeStatus.firstYdata - 50 ||
                e.clientY >= this.changeStatus.lastYdata + 50
                  ? 0
                  : e.clientY - this.originY;
              // console.log(e.clientY <= this.changeStatus.firstYdata - 50);
              // console.log(e.clientY >= this.changeStatus.lastYdata + 50);
              // console.log("dy: " + dy);
              //originTop + e.clientY - originY
              //원래위치 + e.clientY - originY
              console.log(prevOriginY + " / " + this.originY);

              if (prevOriginY == this.originY) {
                // this.$refs.oneItem.style.transform = "translateY(" + dy + "px)";
              }
              this.$refs.oneItem.style.transform =
                "translateY(" + dy / 7 + "px)";
              this.$emit("changeOrder", this.todoItem, e.clientY, this.index);
            }
          }
        }, 100),
      );
      document.addEventListener(
        "mouseup",
        e => {
          this.$refs.oneItem.style.boxShadow = "none";
          // this.$refs.oneItem.style.position = "relative";
          // this.$refs.oneItem.style.width = "auto";
          // this.$refs.oneItem.style.top = "auto";
          this.$refs.oneItem.style.transform = "translateY(0px)";

          this.isDragging = false;
        },
        { once: true },
      );
      // document.addEventListener(
      //   "mouseup",
      //   e => {
      //     this.$emit("changeOrder", this.todoItem, e.clientY, this.index);
      //     this.$refs.oneItem.style.boxShadow = "none";
      //     // this.$refs.oneItem.style.position = "relative";
      //     this.$refs.oneItem.style.width = "auto";
      //     // this.$refs.oneItem.style.top = "auto";

      //     this.isDragging = false;
      //   },
      //   { once: true },
      // );
    },
  },
});
</script>

<style></style>
