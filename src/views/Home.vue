<template>
	<div>
		<div v-if="showAddTask">
			<!-- can also use v-show -->
			<AddTask @add-task="addTask" />
		</div>

		<Tasks
			@toggle-reminder="toggleReminder"
			@delete-task="deleteTask"
			:tasks="this.tasks"
		/>
	</div>
</template>

<script>
import AddTask from "../components/AddTask.vue";
import Tasks from "../components/Tasks.vue";

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
		return { tasks: [] };
	},
	methods: {
		async addTask(task) {
			const res = await fetch("api/tasks", {
				method: "POST",
				headers: {
					"Content-type": "application/json",
				},
				body: JSON.stringify(task),
			});

			const data = await res.json();

			this.tasks = [...this.tasks, data];
		},
		async deleteTask(id) {
			if (confirm("Are you sure?")) {
				const res = await fetch(`api/tasks/${id}`, {
					method: "DELETE",
					headers: {
						"Content-type": "application/json",
					},
				});
				res.status === 200
					? (this.tasks = this.tasks.filter((task) => task.id !== id))
					: alert("Error deleting task");
			}
		},
		async toggleReminder(id) {
			const taskToToggle = await this.fetchTask(id);
			const updTask = { ...taskToToggle, reminder: !taskToToggle.reminder };

			const res = await fetch(`api/tasks/${id}`, {
				method: "PUT",
				headers: {
					"Content-type": "application/json",
				},
				body: JSON.stringify(updTask),
			});

			const data = await res.json();

			this.tasks = this.tasks.map((task) =>
				task.id === id ? { ...task, reminder: data.reminder } : task
			);
		},
		async fetchTasks() {
			const res = await fetch("api/tasks");
			const data = res.json();
			return data;
		},
		async fetchTask(id) {
			const res = await fetch(`api/tasks/${id}`);
			const data = res.json();
			return data;
		},
	},
	async created() {
		this.tasks = await this.fetchTasks();
	},
};
</script>
