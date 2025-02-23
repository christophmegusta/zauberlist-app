<script>
  export let title;
  export let todos;
  export let moveTodo;
	export let moveTodoBack;
  export let listIndex;

  function handleMove(todoId) {
    moveTodo(todoId);
  }

  function handleMoveBack(todoId) {
    moveTodoBack(todoId);
  }

  function handleDragStart(event, todo) {
    event.dataTransfer.setData('text/plain', JSON.stringify({
      todoId: todo.id,
      sourceListIndex: listIndex
    }));
  }

  function handleDragOver(event) {
    event.preventDefault();
    event.currentTarget.classList.add('drag-over');
  }

  function handleDragLeave(event) {
    event.currentTarget.classList.remove('drag-over');
  }

  function handleDrop(event) {
    event.preventDefault();
    event.currentTarget.classList.remove('drag-over');
    const data = JSON.parse(event.dataTransfer.getData('text/plain'));
    const { todoId, sourceListIndex } = data;
    
    if (sourceListIndex !== listIndex) {
      // Move directly to target list
      if (sourceListIndex < listIndex) {
        moveTodo(todoId, listIndex);
      } else {
        moveTodoBack(todoId, listIndex);
      }
    }
  }
</script>

<div 
  class="list"
  on:dragover={handleDragOver}
  on:dragleave={handleDragLeave}
  on:drop={handleDrop}
>
  <h2 class="list-title">{title}</h2>
  {#each todos as todo}
    <div 
      class="todo"
      draggable="true"
      on:dragstart={(e) => handleDragStart(e, todo)}
    >
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
    cursor: grab;
    transition: transform 0.2s ease;
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

  .drag-over {
    background-color: #ffffff40;
    transition: background-color 0.2s ease;
  }

  .todo:active {
    cursor: grabbing;
  }

  .todo:hover {
    transform: scale(1.01);
  }
</style>
