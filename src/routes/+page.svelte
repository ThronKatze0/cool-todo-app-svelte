<script lang="ts">
	import { Modal, Toast, getModalStore, initializeStores } from '@skeletonlabs/skeleton';
	import type { ModalSettings, ModalStore, ToastStore } from '@skeletonlabs/skeleton';
	import { getToastStore } from '@skeletonlabs/skeleton';
	initializeStores();

	enum ToastType {
		SUCCESS,
		WARNING,
		ERROR
	}

	class Todo {
		name: string;
		checked: boolean;
		id: number;
		editModal: ModalSettings;

		constructor(name: string) {
			this.name = name;
			this.checked = false;
			this.id = Math.floor(Math.random() * 100);
			this.editModal = {
				type: 'prompt',
				title: 'Edit Todo',
				body: 'You can edit your Todo below',
				value: '',
				valueAttr: { type: 'text', minlength: 3, maxlength: 10, required: true },
				response: (r: string) => {
					if (r && r != this.name) {
						triggerToast(
							'Todo name succesfully changed from ' + this.name + ' to ' + r,
							ToastType.SUCCESS
						);
						this.name = r;
						todos = todos;
					} else {
						triggerToast('Todo not edited', ToastType.WARNING);
					}
				}
			};
		}
	}

	let inputName: string;
	const modalStore: ModalStore = getModalStore();
	const toastStore: ToastStore = getToastStore();

	let todos: Array<Todo> = [
		new Todo('Todo1'),
		new Todo('Todo1'),
		new Todo('Todo1'),
		new Todo('Todo1')
	];

	let currentFilterMode = 'all';

	function addTodo(): void {
		todos = [...todos, new Todo(inputName)];
		inputName = '';
	}

	function triggerToast(message: string, type: ToastType): void {
		let styleString: string;
		switch (type) {
			case ToastType.SUCCESS:
				styleString = 'variant-filled-primary';
				break;
			case ToastType.WARNING:
				styleString = 'variant-filled-warning';
				break;
			case ToastType.ERROR:
				styleString = 'variant-filled-error';
				break;
		}
		toastStore.trigger({
			message: message,
			background: styleString
		});
	}

	function filterTodos() {
		switch (currentFilterMode) {
			case 'checked':
				triggerToast('Checked todos', ToastType.SUCCESS);
				return todos.filter((todo) => todo.checked);
			case 'unchecked':
				triggerToast('Unchecked todos', ToastType.SUCCESS);
				return todos.filter((todo) => !todo.checked);
			default:
				triggerToast('All todos', ToastType.SUCCESS);
				return todos;
		}
	}
</script>

<Toast />
<Modal />
<h1 class="h1 text-center mt-8">Todo app</h1>
<form class="flex flex-row justify-center mt-16">
	<input
		class="input w-1/4"
		title="Todo Name"
		type="text"
		placeholder="Todo"
		bind:value={inputName}
	/>
	<button type="submit" class="btn variant-filled ml-3" on:click={addTodo}>Submit</button>
</form>
<form class="flex flex-row justify-center mt-4">
	<select
		class="select w-1/6"
		bind:value={currentFilterMode}
		on:change={() => {
			todos = todos;
		}}
	>
		<option value="all">All</option>
		<option value="checked">Checked</option>
		<option value="unchecked">Unchecked</option>
	</select>
	<button
		class="btn variant-filled ml-3"
		on:click={() => {
			todos = [];
		}}>Clear</button
	>
</form>

<div class="grid grid-cols-3 gap-5 mx-20 mt-16">
	{#each filterTodos() as todo, i}
		<div class="card card-hover">
			<header class="card-header">
				<div class="flex flex-row justify-between">
					{#if todo.checked}
						<h2 class="h2 line-through">{todo.name}</h2>
						<input
							class="checkbox p-3.5"
							type="checkbox"
							checked
							on:click={() => {
								todo.checked = !todo.checked;
								triggerToast(todo.name + ' succesfully unchecked', ToastType.SUCCESS);
							}}
						/>
					{:else}
						<h2 class="h2">{todo.name}</h2>
						<input
							class="checkbox p-3.5"
							type="checkbox"
							on:click={() => {
								todo.checked = !todo.checked;
								triggerToast(todo.name + ' succesfully checked', ToastType.SUCCESS);
							}}
						/>
					{/if}
				</div>
			</header>
			<section class="p-4">
				<div class="flex flex-row justify-end">
					<button
						type="submit"
						class="btn variant-filled mr-3"
						on:click={() => {
							triggerToast(todo.name + ' succesfully deleted', ToastType.SUCCESS);
							todos.splice(i, 1);
							todos = todos;
						}}>Delete</button
					>
					<button
						type="submit"
						class="btn variant-filled"
						on:click={() => {
							todo.editModal.value = todo.name;
							modalStore.trigger(todo.editModal);
						}}>Edit</button
					>
				</div>
			</section>
		</div>
	{/each}
</div>
