<template>
  <BaseDialog
    class="due-date-dialog"
    v-model="showDialog"
    v-if="todo"
    no-padding
  >
    <v-date-picker
      color="primary"
      full-width
      flat
      :value="numberToCode(todo.dueTime)"
      @change="setDueDate($event)"
      class="rounded-0 due-date-dialog__due-date-picker"
    />
    <div class="pa-2 pt-0 d-flex justify-space-between">
      <v-btn
        depressed
        :color="isDueToDate('today') ? 'primary' : ''"
        @click="setDueDate('today')"
        width="calc(50% - 4px)"
      >
        Today
      </v-btn>
      <v-btn
        depressed
        :color="isDueToDate('tomorrow') ? 'primary' : ''"
        @click="setDueDate('tomorrow')"
        width="calc(50% - 4px)"
      >
        Tomorrow
      </v-btn>
    </div>
    <template v-slot:left-buttons>
      <v-btn v-if="todo.dueDate" text @click="clearDueDate" color="red">
        Remove due date
      </v-btn>
    </template>
    <template v-slot:right-buttons>
      <v-btn text @click="setDialogOpened(false)">
        OK
      </v-btn>
    </template>
  </BaseDialog>
</template>

<script lang="ts">
// utils
import { Vue, Component, Prop } from "@/utils/vue-imports";
import dateUtils from "@/utils/date";

// interfaces
import Todo from "@/interfaces/entities/todo";

// store modules
import { todosModule } from "@/store";

// components
import BaseDialog from "@/components/bases/BaseDialog.vue";

// component
@Component({
  name: "DueDateDialog",
  components: {
    BaseDialog,
  },
})
export default class DueDateDialog extends Vue {
  // props

  @Prop() readonly todo!: Todo;

  // data
  private showDialog = false;

  // public methods
  public setDialogOpened(open: boolean) {
    this.showDialog = open;
  }

  // private methods
  private isDueToDate(code: string): boolean {
    if (this.todo.dueDate) {
      return code === dateUtils.numberToCode(this.todo.dueDate);
    } else {
      return false;
    }
  }

  private numberToCode(): string {
    return dateUtils.numberToCode(this.todo.dueDate, true);
  }

  setDueDate(code: string) {
    todosModule.setDueDate({
      todoId: this.todo.id,
      dueDate: dateUtils.codeToNumber(code),
    });
    this.showDialog = false;
  }

  private clearDueDate() {
    todosModule.setDueDate({
      todoId: this.todo.id,
      dueDate: null,
    });
    this.showDialog = false;
  }
}
</script>

<style lang="scss" scoped>
@import "./DueDateDialog.scss";
</style>
