<script>
  import List from './List.svelte';
  import { onMount } from 'svelte';

  let boards = [];
  let currentBoard = null;
  let newTodoTitle = '';

  function createTodo() {
    if (!newTodoTitle) {
      return; // Ignore empty todo titles
    }

    const firstListIndex = currentBoard.lists.findIndex(
      (list) => list.title === currentBoard.lists[0].title
    );

    const newTodo = {
      id: Date.now(),
      title: newTodoTitle,
    };

    currentBoard.lists[firstListIndex].todos.push(newTodo);
    newTodoTitle = ''; // Clear the input field

    boards = [...boards];
    currentBoard = { ...currentBoard };
  }

  // https://images.pexels.com/photos/1496372/pexels-photo-1496372.jpeg
  function createBoard(event, title = null, image = null) {
    const newBoard = {
      id: Date.now(),
      title: title || ("New Board " + (boards.length + 1)),
      backgroundImage:
        image || 'https://images.pexels.com/photos/1420440/pexels-photo-1420440.jpeg', // Add the URL of the image as a string here
      lists: [
        {
          id: Date.now() + 1,
          title: 'To Do',
          todos: [
            { id: Date.now() + 2, title: 'Task 1' },
            { id: Date.now() + 3, title: 'Task 2' },
          ],
        },
        {
          id: Date.now() + 4,
          title: 'In Progress',
          todos: [],
        },
        {
          id: Date.now() + 5,
          title: 'Done',
          todos: [],
        },
      ],
    };

    boards = [...boards, newBoard];
    currentBoard = newBoard;
  }

  function moveTodoHandler(todoId) {
    const currentListIndex = currentBoard.lists.findIndex((list) => {
      return list.todos.find((todo) => todo.id === todoId);
    });

    if (currentListIndex === -1) {
      return; // Todo not found
    }

    const todoIndex = currentBoard.lists[currentListIndex].todos.findIndex(
      (todo) => todo.id === todoId
    );

    const movedTodo = currentBoard.lists[currentListIndex].todos.splice(
      todoIndex,
      1
    )[0];

    const nextListIndex =
      currentListIndex === currentBoard.lists.length - 1
        ? currentListIndex
        : currentListIndex + 1;

    currentBoard.lists[nextListIndex].todos.push(movedTodo);

    boards = [...boards];
    currentBoard = { ...currentBoard };
  }

  function moveTodoBack(todoId) {
    const currentListIndex = currentBoard.lists.findIndex((list) => {
      return list.todos.find((todo) => todo.id === todoId);
    });

    if (currentListIndex === -1) {
      return; // Todo not found
    }

    const todoIndex = currentBoard.lists[currentListIndex].todos.findIndex(
      (todo) => todo.id === todoId
    );

    const movedTodo = currentBoard.lists[currentListIndex].todos.splice(
      todoIndex,
      1
    )[0];

    const prevListIndex =
      currentListIndex === 0 ? 0 : currentListIndex - 1;

    currentBoard.lists[prevListIndex].todos.push(movedTodo);

    boards = [...boards];
    currentBoard = { ...currentBoard };
  }

  // onMount((e) => {
  //   createBoard(e);
  //   createBoard(e, null, 'https://images.pexels.com/photos/1496372/pexels-photo-1496372.jpeg');
  //   createBoard(e, null, 'https://images.pexels.com/photos/628281/pexels-photo-628281.jpeg');
  //   createBoard(e, null, 'https://images.pexels.com/photos/220118/pexels-photo-220118.jpeg');
  //   createBoard(e, null, 'https://images.pexels.com/photos/1624496/pexels-photo-1624496.jpeg');
    
  // });
</script>

<div class="app">
  {#if currentBoard}
    <div class="board" style="background-image: url({currentBoard.backgroundImage})">
      <div class="controls-container">
        <div class="board-create">
          <button on:click={createBoard}>+</button>
        </div>

        <div class="board-select">
          <select bind:value={currentBoard} class="board-select-dropdown">
            {#each boards as board}
              <option value={board}>{board.title}</option>
            {/each}
          </select>
        </div>

        <div class="add-todo-form">
          <form on:submit|preventDefault={createTodo}>
            <div class="input-container">
              <input type="text" class="todo-input" bind:value={newTodoTitle} placeholder="Enter todo title" />
              <button type="submit">Add</button>
            </div>
          </form>
        </div>
      </div>

      <div class="lists-container">
        {#each currentBoard.lists as list}
          <List title={list.title} todos={list.todos} moveTodo={moveTodoHandler} moveTodoBack={moveTodoBack} />
        {/each}
      </div>
    </div>
  {:else}
    <p>No board created.</p>
    <button on:click={createBoard}>Create Board</button>
  {/if}
</div>

<style>
  .app {
    padding: 8px;
    filter: drop-shadow(5px 5px 0.5rem black);
  }

  .board-create {
    margin-top: -32px;

  }
  .board-create button {
    background-color: #00000020;
    color:white;
  }

  .board-select {
    margin-bottom: 16px;
  }

  .board-select-dropdown {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: #00000020;
    color: white;
  }

  .board {
    position: relative;
    background-size: cover;
    padding: 16px;
    padding-top: 18px;
    border-radius: 8px;
  }

  .add-todo-form {
    top: 16px;
    left: 50%;
    width: calc(100% - 32px);
    margin-bottom: 16px;
    margin-left: 8px;
    z-index: 1;
  }

  .controls-container {
    display: flex;
    align-items: center;
  }

  .input-container {
    display: flex;
    align-items: center;
  }

  .input-container .todo-input {
    flex: 1;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: #00000020;
    color: white;
  }

  .input-container .todo-input::placeholder {
    color: white;
  }

  .input-container button {
    height: 25px;
    padding: 0 10px;
    background-color: #ffffff10;
    border: none;
    border-radius: 4px;
    color: #fff;
    cursor: pointer;
		position:relative;
		
		right:55px;
		margin:0;
		margin-bottom:7px;
		margin-right:-30px;
  }

  .lists-container {
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
  }

  .board .list {
    flex: 1;
    margin-right: 16px;
    margin-bottom: 16px;
    background-color: #ffffff20;
  }

  button {
    margin-top: 16px;
  }
</style>