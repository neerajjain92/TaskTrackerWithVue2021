<template>
  <AddTask v-show="showAddTask" @add-task="addTask" />
  <Tasks
    @toggle-reminder="toggleReminder"
    @delete-task="deleteTask"
    :tasks="tasks"
  />
</template>
<script>
import Tasks from "../components/Tasks";
import AddTask from "../components/AddTask";
export default {
  name: "Home",
  components: {
    Tasks,
    AddTask,
  },
  props: {
    showAddTask: Boolean,
  },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async deleteTask(id) {
      if (confirm("Are you sure")) {
        const res = await fetch(`api/tasks/${id}`, { method: "DELETE" });

        res.status === 200
          ? (this.tasks = this.tasks.filter((task) => task.id !== id))
          : alert(`Error deleting Task ${id}`);
      }
    },
    async toggleReminder(id) {
      const taskToBeToggled = await this.fetchTask(id);
      const updTask = {
        ...taskToBeToggled,
        reminder: !taskToBeToggled.reminder,
      };

      const res = await fetch(`api/tasks/${id}`, {
        method: "PUT",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify(updTask),
      });

      const data = await res.json();

      // // '...' spread Syntax
      this.tasks = this.tasks.map((task) =>
        task.id === id ? { ...task, reminder: data.reminder } : task
      );
    },

    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`);
      return await res.json();
    },

    async fetchTasks() {
      const res = await fetch(`api/tasks`);
      const data = await res.json();
      return data;
    },

    async addTask(newTask) {
      console.log("New Task to be added ", newTask);
      const res = await fetch("api/tasks", {
        method: "POST",
        headers: {
          "Content-type": "application/json",
        },
        body: JSON.stringify(newTask),
      });

      const data = await res.json();
      // '...' is a spread syntax it takes the iterable (example an array)
      // and expands it into individual elements
      this.tasks = [...this.tasks, data];
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>