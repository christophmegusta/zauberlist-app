<script>
  export let title;
  export let todos;
  export let moveTodo;
	export let moveTodoBack;

  function handleMove(todoId) {
    moveTodo(todoId);
  }

  function handleMoveBack(todoId) {
    moveTodoBack(todoId);
  }
</script>

<div class="list">
  <h2 class="list-title">{title}</h2>
  {#each todos as todo}
    <div class="todo">
      <div class="todo-content">
        <div class="todo-checkbox">
          <input type="checkbox" on:change={() => handleMove(todo.id)} />
        </div>
        <p>{todo.title}</p>
        <button on:click={() => handleMoveBack(todo.id)}>X</button>
      </div>
    </div>
  {/each}
</div>

<style>
  .list {
    flex: 1;
    margin-right: 16px;
		background-color:#ffffff20;
  }

  .todo {
    display: flex;
    align-items: center;
    margin-bottom: 8px;
		margin-left:8px;
  }
	
	.list-title {
		margin:10px;
		color:white;
	}

  .todo-checkbox {
    margin-top:8px;
    margin-right: 8px;
  }

  .todo-content {
    display: flex;
    align-items: center;
    background-color: #f9f9f9;
    padding: 8px;
    border-radius: 4px;
		width:100%;
		margin-right:8px;
  }

  .todo-content p {
    margin: 0;
    flex: 1;
  }

  .todo-content button {
    margin-left: 8px;
    margin-top:8px;
    color: #ccc;
  }
</style>
