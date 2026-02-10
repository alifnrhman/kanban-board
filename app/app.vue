<script setup lang="ts">
import draggable from "vuedraggable";

const assignedUsers = ref(["All Users", "Joko", "Budi", "Siti", "Andi", "Rina"]);
const value = ref("All Users");
const boards = ref([
	{
		id: 1,
		title: "To-do List",
		items: [
			{ id: 1, title: "Menggulingkan pemerintah", description: "Karena sudah tidak adil terhadap rakyat", isComplete: false, assignedTo: "Joko" },
			{ id: 2, title: "Membakar hutan", description: "Untuk membuka lahan baru dan bikin tambang", isComplete: false, assignedTo: "Budi" },
		],
	},
	{
		id: 2,
		title: "In Progress",
		items: [
			{ id: 4, title: "Menipu rakyat", description: "Supaya bisa terus berkuasa", isComplete: false, assignedTo: "Andi" },
			{
				id: 7,
				title: "Merusak sistem pendidikan",
				description: "Agar generasi muda tidak kritis dan mudah dikendalikan",
				isComplete: false,
				assignedTo: "Andi",
			},
		],
	},
	{
		id: 3,
		title: "Completed",
		items: [
			{ id: 3, title: "Mencuri uang negara", description: "Supaya cepat kaya dan bisa beli rumah mewah", isComplete: false, assignedTo: "Siti" },
			{
				id: 6,
				title: "Menghancurkan lingkungan",
				description: "Merusak alam demi keuntungan pribadi",
				isComplete: true,
				assignedTo: "Rina",
			},
		],
	},
]);

// Save the state to localStorage whenever it changes
watch(
	boards,
	(newBoards) => {
		localStorage.setItem("boards", JSON.stringify(newBoards));
	},
	{ deep: true },
);

const filteredBoards = computed(() => {
	if (value.value === "All Users") {
		return boards.value;
	}
	return boards.value.map((board) => {
		return {
			...board,
			items: board.items.filter((item) => item.assignedTo === value.value),
		};
	});
});

const addTask = (boardId: number) => {
	const title = prompt("Enter task title:");
	if (!title) return;
	const description = prompt("Enter task description:") || "";
	const assignedTo = prompt("Enter assigned user:") || "Unassigned";

	const board = boards.value.find((b) => b.id === boardId);
	if (board) {
		const newTask = {
			id: Date.now(),
			title,
			description,
			isComplete: false,
			assignedTo,
		};
		board.items.push(newTask);
	}
};
</script>
<template>
	<div class="px-20 py-10 space-y-2">
		<h1 class="text-3xl font-bold mb-6">Task Management Board</h1>

		<USelectMenu
			label="Assigned Users"
			v-model="value"
			:items="assignedUsers" />

		<div class="gap-5 grid grid-cols-3">
			<div
				v-for="(board, index) in filteredBoards"
				:key="index">
				<h2 class="text-lg font-semibold mb-4">{{ board.title }}</h2>
				<draggable
					v-model="board.items"
					item-key="id"
					tag="ul"
					group="boards"
					class="flex flex-col gap-3">
					<template #item="{ element }">
						<li
							class="p-4 border border-neutral-200 rounded cursor-pointer select-none hover:border-neutral-400 transition-all duration-200"
							:name="element.title">
							<div class="flex flex-col gap-2">
								<h3 class="font-medium">{{ element.title }}</h3>
								<p class="text-sm text-neutral-600">{{ element.description }}</p>
								<p class="text-sm text-neutral-600 text-right">
									Assigned to: <span class="font-semibold">{{ element.assignedTo }}</span>
								</p>
							</div>
						</li>
					</template>
					<template #footer>
						<UButton
							@click="addTask(board.id)"
							block
							variant="outline">
							+ Add Task
						</UButton>
					</template>
				</draggable>
			</div>
		</div>
	</div>
</template>
<style scoped>
.drop-target {
	border-bottom: 3px dashed green;
}
</style>
