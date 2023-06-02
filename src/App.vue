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
        <transition-group name="list" tag="ul" style="transition: 5s">
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
            @readyYdata="resetYdata"
            :changeStatus="changeStatus"
            @resetChangeStatus="resetChangeStatus"
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

const STORAGE_KEY = "vue-todo-ts-v1";
const storage = {
  save(todoItems: Todo[]) {
    const parsed = JSON.stringify(todoItems);
    localStorage.setItem(STORAGE_KEY, parsed);
  },
  fetch(): Todo[] {
    const todoItems =
      localStorage.getItem(STORAGE_KEY) ||
      JSON.stringify([
        { title: "왼쪽 하트버튼을 클릭해보세요", done: false, no: 0 },
        { title: "글자영역을 클릭하여 내용을 수정하세요", done: false, no: 1 },
        {
          title: "오른쪽 Bar 버튼을 위아래로 드래그하여 위치를 변경하세요",
          done: false,
          no: 2,
        },
      ]);
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

export interface ChangeStatus {
  status: boolean;
  firstYdata: number;
  lastYdata: number;
}

export default Vue.extend({
  components: { TodoInput, TodoListItem },
  data() {
    return {
      todoText: "",
      todoItems: [] as Todo[],
      changeStatus: {
        status: false,
        firstYdata: 0,
        lastYdata: 0,
      } as ChangeStatus,
    };
  },
  methods: {
    updateTodoText(value: string) {
      let xx = value.replace(/\s{2,}/gi, " ");
      this.todoText = xx;
    },
    addTodoItem() {
      const value = this.todoText;
      if (value !== "") {
        this.todoItems.push({
          title: value,
          done: false,
          no:
            this.todoItems.length > 0
              ? Math.max(...this.todoItems.map(v => v.no)) + 1
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
    // changeOrderTodoItem(todoItem: Todo, targetH: number, index: number) {
    //   let targetIndex = 0;

    //   console.log(todoItem.ydata + " / " + targetH);

    //   if (todoItem.ydata! < targetH) {
    //     // this.todoItems.forEach((v, i) => {
    //     //   if (i != index && v.ydata! < targetH) {
    //     //     targetIndex = i;
    //     //     return;
    //     //   }
    //     // });
    //     for (let i = 0; i < this.todoItems.length - 1; i++) {
    //       // console.log(this.todoItems[i].ydata);
    //       // console.log(this.todoItems[i + 1].ydata);

    //       if (
    //         targetH >= this.todoItems[i].ydata! &&
    //         targetH < this.todoItems[i + 1].ydata!
    //       ) {
    //         targetIndex = i;
    //         // console.log("==targetindex====" + targetIndex);
    //       } else if (
    //         i == this.todoItems.length - 2 &&
    //         targetH >= this.todoItems[i + 1].ydata!
    //       ) {
    //         targetIndex = i + 1;
    //         // console.log("제일끝일떼 " + targetIndex);
    //       }
    //     }
    //   } else {
    //     for (let i = 0; i < this.todoItems.length; i++) {
    //       // console.log(this.todoItems[i].ydata);
    //       if (targetH < this.todoItems[i].ydata!) {
    //         targetIndex = i;
    //         // console.log("==targetindex====" + targetIndex);
    //         break;
    //       }
    //     }
    //   }

    //   console.log(index + "<- index ");
    //   console.log(targetIndex + "<- targetIndex ");

    //   let prevItems = [];
    //   let changeItems = [];
    //   let betweenItem = [];
    //   let targetItem = [];
    //   let restItem = [];
    //   if (index != targetIndex) {
    //     if (index < targetIndex) {
    //       prevItems = this.todoItems.slice(0, index);
    //       changeItems = this.todoItems.slice(index, index + 1);
    //       betweenItem = this.todoItems.slice(index + 1, targetIndex);
    //       targetItem = this.todoItems.slice(targetIndex, targetIndex + 1);
    //       restItem = this.todoItems.slice(targetIndex + 1);

    //       this.todoItems = [
    //         ...prevItems,
    //         ...betweenItem,
    //         ...targetItem,
    //         ...changeItems,
    //         ...restItem,
    //       ];
    //     } else {
    //       prevItems = this.todoItems.slice(0, targetIndex); // 1,2
    //       changeItems = this.todoItems.slice(index, index + 1); // 5
    //       betweenItem = this.todoItems.slice(targetIndex, index); // 4
    //       // targetItem = this.todoItems.slice(targetIndex, targetIndex + 1); //
    //       restItem = this.todoItems.slice(index + 1);

    //       this.todoItems = [
    //         ...prevItems,
    //         ...changeItems,
    //         ...betweenItem,
    //         // ...targetItem,
    //         ...restItem,
    //       ];
    //     }

    //     this.todoItems.map((v, i) => {
    //       v.no = i;
    //     });
    //     this.resetYdata();
    //     storage.save(this.todoItems);
    //   }
    // },
    // changeOrderTodoItem(todoItem: Todo, targetH: number, index: number) {
    //   let targetIndex = 0;

    //   console.log(todoItem.ydata + " / " + targetH);

    //   if (todoItem.ydata! < targetH) {
    //     // this.todoItems.forEach((v, i) => {
    //     //   if (i != index && v.ydata! < targetH) {
    //     //     targetIndex = i;
    //     //     return;
    //     //   }
    //     // });
    //     for (let i = 0; i < this.todoItems.length - 1; i++) {
    //       // console.log(this.todoItems[i].ydata);
    //       // console.log(this.todoItems[i + 1].ydata);

    //       if (
    //         targetH >= this.todoItems[i].ydata! &&
    //         targetH < this.todoItems[i + 1].ydata!
    //       ) {
    //         targetIndex = i;
    //         // console.log("==targetindex====" + targetIndex);
    //       } else if (
    //         i == this.todoItems.length - 2 &&
    //         targetH >= this.todoItems[i + 1].ydata!
    //       ) {
    //         targetIndex = i + 1;
    //         // console.log("제일끝일떼 " + targetIndex);
    //       }
    //     }
    //   } else {
    //     for (let i = 0; i < this.todoItems.length; i++) {
    //       // console.log(this.todoItems[i].ydata);
    //       if (targetH < this.todoItems[i].ydata!) {
    //         targetIndex = i;
    //         // console.log("==targetindex====" + targetIndex);
    //         break;
    //       }
    //     }
    //   }

    //   console.log(index + "<- index ");
    //   console.log(targetIndex + "<- targetIndex ");

    //   let prevItems = [];
    //   let changeItems = [];
    //   let betweenItem = [];
    //   let targetItem = [];
    //   let restItem = [];
    //   if (index != targetIndex) {
    //     if (index < targetIndex) {
    //       prevItems = this.todoItems.slice(0, index);
    //       changeItems = this.todoItems.slice(index, index + 1);
    //       betweenItem = this.todoItems.slice(index + 1, targetIndex);
    //       targetItem = this.todoItems.slice(targetIndex, targetIndex + 1);
    //       restItem = this.todoItems.slice(targetIndex + 1);

    //       this.todoItems = [
    //         ...prevItems,
    //         ...betweenItem,
    //         ...targetItem,
    //         ...changeItems,
    //         ...restItem,
    //       ];
    //     } else {
    //       prevItems = this.todoItems.slice(0, targetIndex); // 1,2
    //       changeItems = this.todoItems.slice(index, index + 1); // 5
    //       betweenItem = this.todoItems.slice(targetIndex, index); // 4
    //       // targetItem = this.todoItems.slice(targetIndex, targetIndex + 1); //
    //       restItem = this.todoItems.slice(index + 1);

    //       this.todoItems = [
    //         ...prevItems,
    //         ...changeItems,
    //         ...betweenItem,
    //         // ...targetItem,
    //         ...restItem,
    //       ];
    //     }

    //     this.todoItems.map((v, i) => {
    //       v.no = i;
    //     });
    //     this.resetYdata();
    //     storage.save(this.todoItems);
    //   }
    // },
    changeOrderTodoItem(todoItem: Todo, targetH: number, index: number) {
      let targetIndex = 0;

      // console.log(todoItem.ydata + " / " + targetH);

      let currentIndex = 0;
      this.todoItems.forEach((v, i) => {
        if (todoItem.ydata === v.ydata) {
          currentIndex = i;
          return;
        }
      });

      const nextIndex =
        currentIndex != this.todoItems.length - 1 ? currentIndex + 1 : 0;
      if (nextIndex && targetH >= this.todoItems[nextIndex].ydata!) {
        [
          this.todoItems[currentIndex].title,
          this.todoItems[currentIndex].no,
          this.todoItems[currentIndex].done,
          this.todoItems[nextIndex].title,
          this.todoItems[nextIndex].no,
          this.todoItems[nextIndex].done,
        ] = [
          this.todoItems[nextIndex].title,
          this.todoItems[nextIndex].no,
          this.todoItems[nextIndex].done,
          this.todoItems[currentIndex].title,
          this.todoItems[currentIndex].no,
          this.todoItems[currentIndex].done,
        ];
        this.changeStatus.status = true;
        this.resetYdata();
        storage.save(this.todoItems);
      }

      const prevIndex = currentIndex != 0 ? currentIndex - 1 : -1;
      if (prevIndex != -1 && targetH <= this.todoItems[prevIndex].ydata!) {
        [
          this.todoItems[currentIndex].title,
          this.todoItems[currentIndex].no,
          this.todoItems[currentIndex].done,
          this.todoItems[prevIndex].title,
          this.todoItems[prevIndex].no,
          this.todoItems[prevIndex].done,
        ] = [
          this.todoItems[prevIndex].title,
          this.todoItems[prevIndex].no,
          this.todoItems[prevIndex].done,
          this.todoItems[currentIndex].title,
          this.todoItems[currentIndex].no,
          this.todoItems[currentIndex].done,
        ];
        this.changeStatus.status = true;
        this.resetYdata();
        storage.save(this.todoItems);
      }
      // this.todoItems.map((v, i) => {
      //   v.no = i;
      // });
    },
    resetChangeStatus(status: boolean) {
      this.changeStatus.status = status;
    },
    resetYdata() {
      setTimeout(() => {
        // console.log("=======resetYdata 호출22222!!!!");
        // console.log(this.todoItems);
        const itemNodeList = document.querySelectorAll(
          "main ul li",
        ) as NodeList;
        itemNodeList.forEach((v, i) => {
          let target = v as HTMLLIElement;
          if (this.todoItems[i]) {
            this.todoItems[i].ydata = target.offsetTop;
          }
          if (i === 0) this.changeStatus.firstYdata = this.todoItems[i].ydata!;
          if (i === this.todoItems.length - 1) {
            this.changeStatus.lastYdata = this.todoItems[i].ydata!;
          }
        });
      }, 100);
    },
  },
  created() {
    this.fetchTodoItems();
  },
  mounted() {
    this.$nextTick(function () {
      this.resetYdata();
      storage.save(this.todoItems);
    });
  },
});
</script>

<style lang="scss">
@import "@/scss/common";
</style>
