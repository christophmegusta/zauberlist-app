<script>
  export let todo;
  export let moveTodo;
</script>

<div class="todo">
  <div class="checkbox-container">
    <input type="checkbox" on:change={() => moveTodo(todo.id)} />
  </div>
  <p>{todo.title}</p>
  <button class="move-button" on:click={() => moveTodoBack(todo.id)}>
    Move Back
  </button>
</div>

<style>
  .todo {
    display: flex;
    align-items: center;
    padding: 8px;
    background-color: #f4f4f4;
    border-radius: 4px;
    margin-bottom: 8px;
  }

  .checkbox-container {
    margin-right: 8px;
  }

  .move-button {
    margin-left: auto;
    background-color: #f44336;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 4px 8px;
    cursor: pointer;
  }
</style>
