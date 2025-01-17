<template>
  <Swipeout
    :left-actions="swipeoutLeftActions"
    :right-actions="swipeoutRightActions"
    :enable="!expanded"
    :class="{
      'todo-card': true,
      'todo-card--checked': todo.checked,
    }"
  >
    <div
      @contextmenu.prevent="rightClick"
      v-touch:touchhold="touchHold"
      class="todo-card__card"
    >
      <div class="d-flex align-center pa-1 todo-card__header">
        <v-simple-checkbox
          :value="todo.checked"
          @input="toggleChecked()"
          :color="customTodoFolder ? customTodoFolder.color : 'primary'"
          class="todo-card__checkbox"
        />
        <div class="todo-card__title mx-1">
          <v-text-field
            v-if="titleInEdit"
            v-model="tempTitleValue"
            flat
            solo
            hide-details
            dense
            placeholder="Task title"
            outlined
            :color="customTodoFolder ? customTodoFolder.color : 'primary'"
            ref="taskTitleInput"
            class="todo-card__title-input"
            @blur="setTitle"
            @contextmenu.stop
            @keypress.enter="unfocus"
          />
          <button
            v-else
            class="py-1 px-3 todo-card__title-text"
            @click="editTitle"
            v-ripple
          >
            {{ todo.title }}
          </button>
        </div>
        <v-btn icon @click="toggleExpandedTodo">
          <v-icon>
            {{ expanded ? "mdi-chevron-up" : "mdi-chevron-down" }}
          </v-icon>
        </v-btn>
        <div v-if="todo.important" class="todo-card__important-indicator" />
      </div>
      <v-expand-transition>
        <div v-if="expanded" class="todo-card__body">
          <v-divider v-if="$isPhone" />
          <TodoSteps
            :steps="todo.steps"
            :color="customTodoFolder ? customTodoFolder.color : 'primary'"
            :todo-id="todo.id"
          />
          <v-divider />
          <v-textarea
            :value="todo.body"
            placeholder="Description"
            no-resize
            hide-details
            flat
            solo
            @blur="setBody($event.target.value)"
            @contextmenu.stop
            class="todo-card__description"
          />
        </div>
      </v-expand-transition>
      <v-divider
        v-if="customTodoFolder || todo.steps.length > 0 || todo.dueDate"
      />
      <TodoCaption :todo="todo" />
    </div>
  </Swipeout>
</template>

<script lang="ts">
// utils
import { Mixins, Component, Prop } from "@/utils/vue-imports";

// interfaces
import Todo from "@/interfaces/entities/todo";
import Folder from "@/interfaces/entities/folder";

// store modules
import { todosModule, foldersModule } from "@/store";

// mixins
import responsiveMixin from "@/mixins/responsive";

// interfaces
import SwipeoutAction from "@/interfaces/logic/swipeoutAction";

// components
import Swipeout from "@/components/atoms/Swipeout/Swipeout.vue";
import TodoSteps from "@/components/atoms/TodoSteps/TodoSteps.vue";
import TodoCaption from "@/components/atoms/TodoCaption/TodoCaption.vue";

// component
@Component({
  name: "TodoCard",
  components: {
    Swipeout,
    TodoSteps,
    TodoCaption,
  },
})
export default class TodoCard extends Mixins(responsiveMixin) {
  // refs
  public $refs!: {
    taskTitleInput: HTMLInputElement;
  };

  // props
  @Prop() readonly todo!: Todo;
  @Prop() readonly expanded!: boolean;

  // data
  private titleInEdit = false;
  private tempTitleValue = "";
  private swipeoutLeftActions: SwipeoutAction[] = [
    {
      icon: "mdi-folder-outline",
      text: "Folder",
      color: "blue",
      onClick: () => {
        this.chooseFolder();
      },
    },
    {
      icon: "mdi-calendar-blank",
      text: "Due date",
      color: "purple",
      onClick: () => {
        this.addDueDate();
      },
    },
  ];
  private swipeoutRightActions: SwipeoutAction[] = [
    {
      icon: "mdi-octagram-outline",
      text: "Important",
      color: "orange",
      onClick: () => {
        this.toggleImportant();
      },
    },
    {
      icon: "mdi-delete-outline",
      text: "Delete",
      color: "red",
      onClick: () => {
        this.removeTodo();
      },
    },
  ];

  // computed
  private get customTodoFolder(): Folder | null {
    return (
      foldersModule.folders.find(
        (folder: Folder) => folder.id === this.todo.customFolderId
      ) || null
    );
  }

  // private methods
  private toggleChecked() {
    todosModule.setChecked({
      todoId: this.todo.id,
      checked: !this.todo.checked,
    });
  }

  private toggleExpandedTodo() {
    this.$emit("expandToggled");
  }

  private chooseFolder() {
    this.$emit("chooseFolder", this.todo);
  }

  private addDueDate() {
    this.$emit("addDueDate", this.todo);
  }

  private toggleImportant() {
    todosModule.setImportant({
      todoId: this.todo.id,
      important: !this.todo.important,
    });
  }

  private removeTodo() {
    todosModule.removeTodo(this.todo.id);
  }

  private editTitle() {
    this.tempTitleValue = this.todo.title;
    this.titleInEdit = true;
    setTimeout(() => {
      this.$refs.taskTitleInput.focus();
    }, 0);
  }

  private setTitle() {
    this.titleInEdit = false;
    if (this.tempTitleValue.length && this.tempTitleValue !== this.todo.title) {
      todosModule.setTitle({
        todoId: this.todo.id,
        title: this.tempTitleValue,
      });
    }
    this.tempTitleValue = "";
  }

  private unfocus() {
    (document.activeElement as HTMLElement).blur();
  }

  private setBody(body: string) {
    todosModule.setBody({
      todoId: this.todo.id,
      body,
    });
  }
}
</script>

<style lang="scss" scoped>
@import "./TodoCard.scss";
</style>
