<script>
  import { writable, get } from 'svelte/store';
  import { onMount } from 'svelte';
  import List from './List.svelte';
  import Peer from 'peerjs';

  const boards = writable([]);
  const currentBoard = writable(null);
  const peerId = writable(null);
  let newTodoTitle = '';

  const peer = new Peer();

  let connectedPeers = [];
  let isUpdating = false;

  peer.on('open', (id) => {
    console.log('My peer ID is: ' + id);
    peerId.set(id);
  });

  peer.on('connection', (connection) => {
    handlePeerConnection(connection);
  });

  function handlePeerConnection(connection) {
    connection.on('open', () => {
      console.log('Connected to peer: ' + connection.peer);
      connectedPeers.push(connection);

      connection.on('close', () => {
        console.log('Disconnected from peer: ' + connection.peer);
        connectedPeers = connectedPeers.filter(
          (peer) => peer.peer !== connection.peer
        );
      });

      connection.on('data', (data) => {
        console.log('Received data from peer: ', data);
        const { type, data: receivedData } = JSON.parse(data);

        if (type === 'updateBoard' && !isUpdating) {
          const storeData = get(currentBoard);
          if (JSON.stringify(receivedData) !== JSON.stringify(storeData)) {
            currentBoard.set(receivedData);
          }
        }
      });
    });
  }

  function createTodo() {
    if (!newTodoTitle) {
      return;
    }

    const { subscribe, update } = currentBoard;
    update((value) => {
      const firstListIndex = value.lists.findIndex(
        (list) => list.title === value.lists[0].title
      );

      const newTodo = {
        id: Date.now(),
        title: newTodoTitle,
      };

      value.lists[firstListIndex].todos.push(newTodo);
      newTodoTitle = '';

      isUpdating = true;
      connectedPeers.forEach((peer) => {
        peer.send(JSON.stringify({ type: 'updateBoard', data: value }));
      });
      isUpdating = false;

      return { ...value };
    });
  }

  function createBoard(event, title = null, image = null) {
    const newBoard = {
      id: Date.now(),
      title: title || `New Board ${$boards.length + 1}`,
      backgroundImage:
        image ||
        'https://images.pexels.com/photos/1420440/pexels-photo-1420440.jpeg',
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

    const { subscribe, update } = boards;
    update((value) => {
      const newBoards = [...value, newBoard];
      isUpdating = true;
      connectedPeers.forEach((peer) => {
        peer.send(JSON.stringify({ type: 'updateBoard', data: newBoards }));
      });
      isUpdating = false;
      return newBoards;
    });

    currentBoard.set(newBoard);
  }

  function moveTodoHandler(todoId, targetListIndex = null) {
    const { subscribe, update } = currentBoard;
    update((value) => {
      const currentListIndex = value.lists.findIndex((list) => {
        return list.todos.find((todo) => todo.id === todoId);
      });

      if (currentListIndex === -1) {
        return value;
      }

      const todoIndex = value.lists[currentListIndex].todos.findIndex(
        (todo) => todo.id === todoId
      );

      const movedTodo = value.lists[currentListIndex].todos.splice(
        todoIndex,
        1
      )[0];

      const nextListIndex = targetListIndex !== null 
        ? targetListIndex
        : (currentListIndex === value.lists.length - 1
          ? currentListIndex
          : currentListIndex + 1);

      value.lists[nextListIndex].todos.push(movedTodo);

      isUpdating = true;
      connectedPeers.forEach((peer) => {
        peer.send(JSON.stringify({ type: 'updateBoard', data: value }));
      });
      isUpdating = false;

      return { ...value };
    });
  }

  function moveTodoBack(todoId, targetListIndex = null) {
    const { subscribe, update } = currentBoard;
    update((value) => {
      const currentListIndex = value.lists.findIndex((list) => {
        return list.todos.find((todo) => todo.id === todoId);
      });

      if (currentListIndex === -1) {
        return value;
      }

      const todoIndex = value.lists[currentListIndex].todos.findIndex(
        (todo) => todo.id === todoId
      );

      const movedTodo = value.lists[currentListIndex].todos.splice(
        todoIndex,
        1
      )[0];

      const prevListIndex = targetListIndex !== null 
        ? targetListIndex 
        : (currentListIndex === 0 ? 0 : currentListIndex - 1);

      value.lists[prevListIndex].todos.push(movedTodo);

      isUpdating = true;
      connectedPeers.forEach((peer) => {
        peer.send(JSON.stringify({ type: 'updateBoard', data: value }));
      });
      isUpdating = false;

      return { ...value };
    });
  }

  onMount(() => {
    createBoard(null);
    createBoard(null, null, 'https://images.pexels.com/photos/1496372/pexels-photo-1496372.jpeg');
    createBoard(null, null, 'https://images.pexels.com/photos/628281/pexels-photo-628281.jpeg');
    createBoard(null, null, 'https://images.pexels.com/photos/220118/pexels-photo-220118.jpeg');
    createBoard(null, null, 'https://images.pexels.com/photos/1624496/pexels-photo-1624496.jpeg');

    const handleBeforeUnload = () => {
      connectedPeers.forEach((peer) => {
        peer.close();
      });
			peer.destroy();
    };

    window.addEventListener('beforeunload', handleBeforeUnload);

    return () => {
      window.removeEventListener('beforeunload', handleBeforeUnload);
    };
  });

  function connectToPeer() {
    const remotePeerId = prompt('Enter the Peer ID to connect:');
    const connection = peer.connect(remotePeerId);
    handlePeerConnection(connection);
  }
</script>

<div class="app">
  {#if $currentBoard}
    <div class="board" style="background-image: url({$currentBoard.backgroundImage})">
      <div class="title-bar">
        <a href="#{$peerId}" on:click={() => navigator.clipboard.writeText($peerId)}>Copy Your Peer ID: {$peerId}</a>
      </div>

      <div class="controls-container">
        <div class="board-create">
          <button on:click={createBoard}>+</button>
        </div>

        <div class="board-select">
          <select bind:value={$currentBoard} class="board-select-dropdown">
            {#each $boards as board}
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

        <div class="connect-peers">
          <button on:click={connectToPeer}>Connect</button>
        </div>
      </div>

      <div class="lists-container">
        {#each $currentBoard.lists as list, index}
          <List 
            title={list.title} 
            todos={list.todos} 
            moveTodo={moveTodoHandler} 
            moveTodoBack={moveTodoBack}
            listIndex={index} 
          />
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

  .title-bar {
    margin-top:-20px;
    padding-top:10px;
    height:10px;
    font-size:6px;
    text-align: right;
    color:#00000090;
  }
  .title-bar a {
    color:#00000090;
    text-decoration: none;
  }
  .title-bar a:hover {
    color:yellow;
    text-decoration: none;
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

  .connect-peers {
    margin-top: -31px;
    margin-left: -12px;
  }

  .connect-peers button {
    border-radius: 4px;
    background-color: #00000020;
    color: white;    
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
    gap: 16px;
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