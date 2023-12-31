<svelte:options immutable={true} />

<script>
  import Button from './Button.svelte';
  import { createEventDispatcher, afterUpdate } from 'svelte';
  import FaRegTrashAlt from 'svelte-icons/fa/FaRegTrashAlt.svelte';
  import { scale, crossfade } from 'svelte/transition';
  import { flip } from 'svelte/animate';
  export let error = null;
  export let isLoading = false;
  export let disabledAdding = false;
  export let disabledRemoving = [];
  export let scrollOnAdd = undefined;

  afterUpdate(() => {
    if (scrollOnAdd) {
      let pos;
      if (scrollOnAdd === 'top') {
        pos = 0;
      }
      if (scrollOnAdd === 'bottom') {
        pos = listDivScrollHeight;
      }
      if (autoscroll) listDiv.scrollTo(0, pos);
      autoscroll = false;
    }
  });

  $: done = todos ? todos.filter((t) => t.completed) : [];
  $: todo = todos ? todos.filter((t) => !t.completed) : [];

  export let todos = null;
  let prevTodos = todos;
  let inputText = '';
  let input, listDiv, autoscroll, listDivScrollHeight;

  const dispatch = createEventDispatcher();

  $: {
    autoscroll = todos && prevTodos && todos.length > prevTodos.length;
    prevTodos = todos;
  }

  export function clearInput() {
    inputText = '';
  }
  export function focusInput() {
    input.focus();
  }

  function handleAddTodo() {
    const isNotCancelled = dispatch(
      'addtodo',
      {
        title: inputText
      },
      {
        cancelable: true
      }
    );
    if (isNotCancelled) {
      inputText = '';
    }
  }

  function handleRemoveTodo(id) {
    dispatch('removetodo', {
      id
    });
  }

  function handleToggleTodo(id, value) {
    dispatch('toggletodo', {
      id,
      value
    });
  }

const [send, receive] = crossfade({
  duration: 400,
  fallback(node) {
    return scale(node, {start: 0.5, duration: 400});
  }
});

</script>

<div class="todo-list-wrapper">
  {#if isLoading}
    <p class="no-items-text">Loading...</p>
  {:else if error}
    <p class="no-items-text">{error}</p>
  {:else if todos}
    <div class="todo-list" bind:this={listDiv}>
      <div bind:offsetHeight={listDivScrollHeight}>
        {#if todos.length === 0}
          <p class="no-items-text">No todos yet</p>
        {:else}
          <div style:display="flex">
            {#each [todo, done] as list, index}
              <div class="list-wrapper">
                <h2>{index === 0 ? 'Todo' : 'Undone'}</h2>
                <ul>
                  {#each list as todo (todo.id)}
                    {@const { id, title, completed } = todo}

                    <li animate:flip>
                      <slot {todo} {handleToggleTodo}>
                        <div in:receive|local={{key: id}} out:send|local={{key: id}} class:completed>
                          <label>
                            <input
                              disabled={disabledRemoving.includes(id)}
                              on:input={(event) => {
                                event.currentTarget.checked = completed;
                                handleToggleTodo(id, !completed);
                              }}
                              type="checkbox"
                              checked={completed}
                            />
                            {title}
                          </label>
                          <button
                            disabled={disabledRemoving.includes(id)}
                            class="remove-todo-button"
                            aria-label="Remove todo: {title}"
                            on:click={() => handleRemoveTodo(id)}
                          >
                            <span style:width="10px" style:display="inline-block">
                              <FaRegTrashAlt />
                            </span>
                          </button>
                        </div>
                      </slot>
                    </li>
                  {/each}
                </ul>
              </div>
            {/each}
          </div>
        {/if}
      </div>
    </div>
  {/if}
  <form class="add-todo-form" on:submit|preventDefault={handleAddTodo}>
    <input
      disabled={disabledAdding}
      bind:this={input}
      bind:value={inputText}
      placeholder="New Todo"
    />
    <Button class="add-todo-button" type="submit" disabled={!inputText || disabledAdding || !todos}
      >Add</Button
    >
  </form>
</div>

<style lang="scss">
  .todo-list-wrapper {
    background-color: #424242;
    border: 1px solid #4b4b4b;
    .no-items-text {
      margin: 0;
      padding: 15px;
      text-align: center;
    }
    .todo-list {
      max-height: 400px;
      overflow: auto;
      .list-wrapper {
        flex: 1;
        padding: 10px;
        h2 {
          margin: 0 0 10px;
        }
        ul {
          padding: 0;
          margin: 0;
          list-style: none;
          li > div {
            margin-bottom: 5px;
            display: flex;
            align-items: center;
            background-color: #303030;
            border-radius: 5px;
            padding: 10px;
            position: relative;
            label {
              cursor: pointer;
              font-size: 18px;
              display: flex;
              align-items: baseline;
              padding-right: 20px;
              input[type='checkbox'] {
                margin: 0 10px 0 0;
                cursor: pointer;
              }
            }
            &.completed > label {
              opacity: 0.5;
              text-decoration: line-through;
            }
            .remove-todo-button {
              border: none;
              background: none;
              padding: 5px;
              position: absolute;
              right: 10px;
              cursor: pointer;
              display: none;
              &:disabled {
                cursor: not-allowed;
                opacity: 0.5;
              }
              :global(svg) {
                fill: #bd1414;
              }
            }
            &:hover {
              .remove-todo-button {
                display: block;
              }
            }
          }
        }
      }
    }
    .add-todo-form {
      padding: 15px;
      background-color: #303030;
      display: flex;
      flex-wrap: wrap;
      border-top: 1px solid #4b4b4b;
      // :global(.add-todo-button) {
      //   background-color: aqua;
      // }
      input {
        flex: 1;
        background-color: #424242;
        border: 1px solid #4b4b4b;
        padding: 10px;
        color: #fff;
        border-radius: 5px;
        margin-right: 10px;
      }
    }
  }
</style>
