<template>
  <div class="todo_wrap">
    <header><h1>TODO LIST</h1></header>
    <main>
      <todo-input
        class="todoInput_wrap"
        :item="todoText"
        @input="updateTodoText"
        @add="addTodoItem"
      ></todo-input>
      <div class="todoList_wrap">
        <transition-group name="list" tag="ul">
          <!-- <ul> -->
          <todo-list-item
            v-for="(todoItem, index) in todoItems"
            :key="todoItem.no"
            :todoItem="todoItem"
            :index="index"
            @toggle="toggleTodoItemComplete"
            @edit="editTodoItem"
            @remove="removeTodoItem"
            @changeOrder="changeOrderTodoItem"
            @readyYdata="readyYdata"
          ></todo-list-item>
          <!-- </ul> -->
        </transition-group>
      </div>
    </main>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import TodoInput from "./components/TodoInput.vue";
import TodoListItem from "./components/TodoListItem.vue";
import $ from "jquery";

const STORAGE_KEY = "vue-todo-ts-v1";
const storage = {
  save(todoItems: Todo[]) {
    const parsed = JSON.stringify(todoItems);
    localStorage.setItem(STORAGE_KEY, parsed);
  },
  fetch(): Todo[] {
    const todoItems = localStorage.getItem(STORAGE_KEY) || "[]";
    const result = JSON.parse(todoItems);
    return result;
  },
};

export interface Todo {
  title: string;
  done: boolean;
  no: number;
  ydata?: number;
}

export default Vue.extend({
  components: { TodoInput, TodoListItem },
  data() {
    return {
      todoText: "",
      todoItems: [] as Todo[],
    };
  },
  methods: {
    updateTodoText(value: string) {
      this.todoText = value;
    },
    addTodoItem() {
      const value = this.todoText;
      if (value !== "") {
        this.todoItems.push({
          title: value,
          done: false,
          no:
            this.todoItems.length > 0
              ? this.todoItems[this.todoItems.length - 1].no + 1
              : 0,
        });
        this.resetYdata();
        storage.save(this.todoItems);
        this.initTodoText();
      }
    },
    initTodoText() {
      this.todoText = "";
    },
    fetchTodoItems() {
      this.todoItems = storage.fetch();
    },
    removeTodoItem(index: number) {
      this.todoItems.splice(index, 1);
      this.resetYdata();
      storage.save(this.todoItems);
    },
    toggleTodoItemComplete(todoItem: Todo, index: number) {
      this.todoItems.splice(index, 1, {
        ...todoItem,
        done: !todoItem.done,
      });
      storage.save(this.todoItems);
    },
    editTodoItem(todoItem: Todo, index: number, title: string) {
      this.todoItems.splice(index, 1, {
        ...todoItem,
        title: title,
      });
      storage.save(this.todoItems);
    },
    changeOrderTodoItem(todoItem: Todo, targetH: number, index: number) {
      let targetIndex = 0;
      // if()
      //만약 targetH의 index와 index가 같다면? -> 막아놈 -> 아래 로직 잘못됨
      //만약 targetH보다 ydata가 크다면?
      console.log(todoItem.ydata + " / " + targetH);

      if (todoItem.ydata! < targetH) {
        // this.todoItems.forEach((v, i) => {
        //   if (i != index && v.ydata! < targetH) {
        //     targetIndex = i;
        //     return;
        //   }
        // });
        for (let i = 0; i < this.todoItems.length - 1; i++) {
          // console.log(this.todoItems[i].ydata);
          // console.log(this.todoItems[i + 1].ydata);

          if (
            targetH >= this.todoItems[i].ydata! &&
            targetH < this.todoItems[i + 1].ydata!
          ) {
            targetIndex = i;
            // console.log("==targetindex====" + targetIndex);
          } else if (
            i == this.todoItems.length - 2 &&
            targetH >= this.todoItems[i + 1].ydata!
          ) {
            targetIndex = i + 1;
            // console.log("제일끝일떼 " + targetIndex);
          }
        }
      } else {
        for (let i = 0; i < this.todoItems.length; i++) {
          // console.log(this.todoItems[i].ydata);
          if (targetH < this.todoItems[i].ydata!) {
            targetIndex = i;
            // console.log("==targetindex====" + targetIndex);
            break;
          }
        }
      }

      console.log(index + "<- index ");
      console.log(targetIndex + "<- targetIndex ");

      let prevItems = [];
      let changeItems = [];
      let betweenItem = [];
      let targetItem = [];
      let restItem = [];
      if (index != targetIndex) {
        if (index < targetIndex) {
          prevItems = this.todoItems.slice(0, index);
          changeItems = this.todoItems.slice(index, index + 1);
          betweenItem = this.todoItems.slice(index + 1, targetIndex);
          targetItem = this.todoItems.slice(targetIndex, targetIndex + 1);
          restItem = this.todoItems.slice(targetIndex + 1);

          this.todoItems = [
            ...prevItems,
            ...betweenItem,
            ...targetItem,
            ...changeItems,
            ...restItem,
          ];
        } else {
          prevItems = this.todoItems.slice(0, targetIndex); // 1,2
          changeItems = this.todoItems.slice(index, index + 1); // 5
          betweenItem = this.todoItems.slice(targetIndex, index); // 4
          // targetItem = this.todoItems.slice(targetIndex, targetIndex + 1); //
          restItem = this.todoItems.slice(index + 1);

          this.todoItems = [
            ...prevItems,
            ...changeItems,
            ...betweenItem,
            // ...targetItem,
            ...restItem,
          ];
        }

        this.todoItems.map((v, i) => {
          v.no = i;
        });
        this.resetYdata();
        storage.save(this.todoItems);
      }
    },
    readyYdata() {
      this.resetYdata();
    },
    resetYdata() {
      const itemNodeList = document.querySelectorAll("main ul li") as NodeList;
      itemNodeList.forEach((v, i) => {
        setTimeout(() => {
          let target = v as HTMLLIElement;
          this.todoItems[i].ydata = target.offsetTop;
          console.log(i + ": " + target.offsetTop);
        }, 100);
      });
    },
  },
  created() {
    this.fetchTodoItems();
  },
  mounted() {
    this.resetYdata();
    storage.save(this.todoItems);
  },
});
</script>

<style lang="scss">
@import "@/scss/common";
</style>
