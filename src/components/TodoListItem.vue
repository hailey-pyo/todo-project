<template>
  <li :class="todoItemClass" ref="oneItem">
    <div class="heart_wrap" @click="toggleItem">
      <span class="check_heart"></span>
    </div>
    <!-- :ref="'editInput' + index" -->
    <span v-if="!edit.status" class="item stop-dragging">
      <span class="title" @click="editTodo" ref="originTitle">
        {{ todoItem.title }}
      </span>
      <i
        class="fa-solid fa-bars"
        @touchstart="touchStart"
        @mousedown="mouseDown"
      ></i>
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
    originTitle: HTMLSpanElement;
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
      scrollHeight: 0,
    };
  },
  computed: {
    todoItemClass(): string | null {
      return this.todoItem.done ? "complete" : null;
    },
    itemHeight(): object {
      return {
        height: this.scrollHeight + "px",
      };
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
        this.$refs.editInput.removeAttribute("autofocus");
      }, 100);
      setTimeout(() => {
        const textarea = this.$refs.editInput;

        textarea.parentElement!.style.height = this.scrollHeight + "px";
        textarea.style.height = this.scrollHeight + "px";
        textarea.setAttribute("autofocus", "autofocus");
        textarea.focus();
      }, 200);
    },
    editTodoTitle(event: VueEvent.Input<HTMLInputElement>) {
      if (event.inputType == "insertLineBreak") {
        return;
      }
      this.edit.title = event.target.value;
      const textarea = this.$refs.editInput;

      textarea.style.height = "auto";
      textarea.parentElement!.style.height = textarea.scrollHeight + "px";
      textarea.style.height = textarea.scrollHeight + "px";
      this.scrollHeight = textarea.scrollHeight;
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
      this.originY = window.pageYOffset + e.clientY;
      console.log("============1111originY셋팅==========" + this.originY);

      this.isDragging = true;
      this.$emit("readyYdata");

      document.addEventListener(
        "mousemove",
        throttle((e: MouseEvent) => {
          if (this.isDragging) {
            if (this.changeStatus.status) {
              console.log("e.절대값: " + window.pageYOffset + e.clientY);

              this.$refs.oneItem.style.transform = "translateY(0px)";

              this.originY = window.pageYOffset + e.clientY;
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
                window.pageYOffset + e.clientY <=
                  this.changeStatus.firstYdata - 50 ||
                window.pageYOffset + e.clientY >=
                  this.changeStatus.lastYdata + 50
                  ? 0
                  : window.pageYOffset + e.clientY - this.originY;
              // console.log(e.clientY <= this.changeStatus.firstYdata - 50);
              // console.log(e.clientY >= this.changeStatus.lastYdata + 50);
              // console.log("dy: " + dy);
              //originTop + e.clientY - originY
              //원래위치 + e.clientY - originY
              this.$refs.oneItem.style.transform =
                "translateY(" + dy / 7 + "px)";
              this.$emit(
                "changeOrder",
                this.todoItem,
                window.pageYOffset + e.clientY,
                this.index,
              );
            }
          }
        }, 100),
      );
      document.addEventListener(
        "mouseup",
        e => {
          console.log("마우스뗌");

          this.$refs.oneItem.style.boxShadow = "none";
          // this.$refs.oneItem.style.position = "relative";
          // this.$refs.oneItem.style.width = "auto";
          // this.$refs.oneItem.style.top = "auto";
          this.$refs.oneItem.style.transform = "translateY(0px)";

          this.isDragging = false;
        },
        { once: true },
      );
    },
    touchStart(e: TouchEvent) {
      e.preventDefault();
      this.originY =
        window.pageYOffset + Math.round(e.targetTouches[0].clientY);
      console.log("============originY셋팅==========" + this.originY);

      this.isDragging = true;
      this.$emit("readyYdata");

      document.addEventListener(
        "touchmove",
        throttle(e => {
          if (this.isDragging) {
            if (this.changeStatus.status) {
              this.$refs.oneItem.style.transform = "translateY(0px)";

              this.originY =
                window.pageYOffset + Math.round(e.targetTouches[0].clientY);
              console.log("순서변경 완료 originY변경! " + this.originY);
              this.$emit("resetChangeStatus", false);
            } else {
              this.$refs.oneItem.style.boxShadow = "1px 1px 10px rgb(2,2,2)";

              const dy =
                window.pageYOffset + Math.round(e.targetTouches[0].clientY) <=
                  this.changeStatus.firstYdata - 50 ||
                window.pageYOffset + Math.round(e.targetTouches[0].clientY) >=
                  this.changeStatus.lastYdata + 50
                  ? 0
                  : window.pageYOffset +
                    Math.round(e.targetTouches[0].clientY) -
                    this.originY;

              this.$refs.oneItem.style.transform =
                "translateY(" + dy / 7 + "px)";
              this.$emit(
                "changeOrder",
                this.todoItem,
                window.pageYOffset + Math.round(e.targetTouches[0].clientY),
                this.index,
              );
            }
          }
        }, 100),
      );
      document.addEventListener(
        "touchend",
        () => {
          console.log("손가락뗌");

          this.$refs.oneItem.style.boxShadow = "none";
          // this.$refs.oneItem.style.position = "relative";
          // this.$refs.oneItem.style.width = "auto";
          // this.$refs.oneItem.style.top = "auto";
          this.$refs.oneItem.style.transform = "translateY(0px)";

          this.isDragging = false;
        },
        { once: true },
      );
    },
  },
  mounted() {
    this.$nextTick(function () {
      this.scrollHeight = this.$refs.originTitle.scrollHeight;
    });
  },
});
</script>

<style></style>
