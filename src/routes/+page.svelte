<!-- src/components/TodoList.svelte -->
<script>
	import { onMount } from 'svelte'
	import { db } from '$lib/firebase'
	import { addDoc, getDocs, collection, doc, updateDoc } from 'firebase/firestore'

	let todos = []
	let newTodo = ''

	async function fetchTodos() {
		const querySnapshot = await getDocs(collection(db, 'todos'))
		todos = querySnapshot.docs.map((doc) => ({ id: doc.id, ...doc.data() }))
	}

	async function addTodo() {
		if (newTodo.trim() !== '') {
			try {
				const docRef = await addDoc(collection(db, 'todos'), {
					text: newTodo,
					completed: false,
				})
				newTodo = ''
				await fetchTodos()
				console.log('Document written with ID: ', docRef.id)
			} catch (error) {
				console.error('Error adding document: ', error)
			}
		}
	}

	async function toggleCompleted(todoId, isComplete) {
		const todoRef = doc(db, 'todos', todoId)

		try {
			await updateDoc(todoRef, {
				completed: !isComplete,
			})
			console.log(`Updated todo status`)
		} catch (error) {
			console.error('Error updating todo:', error)
		}
		fetchTodos()
	}

	onMount(fetchTodos)
</script>

<section>
	<input bind:value={newTodo} placeholder="Enter a new todo" />
	<button on:click={addTodo}>Add Todo</button>
	<ul>
		{#each todos as todo (todo.id)}
			<li>
				{todo.text}
				<button on:click={() => toggleCompleted(todo.id, todo.completed)}>
					{#if todo.completed}
						Done
					{:else}
						Pending
					{/if}
				</button>
			</li>
		{/each}
	</ul>
</section>
