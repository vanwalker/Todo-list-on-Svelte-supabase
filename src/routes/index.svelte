<script>

  import { goto } from "$app/navigation";
  import Navbar from "$lib/Navbar.svelte";
  import {supabase} from "$lib/supabaseClient"

  import Todo from "$lib/Todo.svelte";
  import { onMount } from "svelte";
  import {user} from "$lib/sessionStore"
  let todos = [];
  let newTask = "";

  user.set(supabase.auth.user())

  supabase.auth.onAuthStateChange((_, session) => {
      user.set(session.user)
  });
  onMount(async () => {
    await getAllTodos();
  });
  const getAllTodos = async () => {
    try {
      let { data, error } = await supabase.from("todos").select("*");
      todos = data;
    } catch (err) {
      console.log(err);
    }
  };
  const addNewTodo = async () => {
    try {
      const { data, error } = await supabase
        .from("todos")
        .insert([{ task: newTask, user_id: $user.id }]);
      await getAllTodos();
      newTask = "";
    } catch (err) {
      console.log(err);
    }
  };
  const updateTodo = async (todo) => {
    try {
      const { data, error } = await supabase
        .from("todos")
        .update({ task: todo.task, isComplete: todo.isComplete })
        .eq("id", todo.id);
      await getAllTodos();
    } catch (err) {
      console.log(err);
    }
  };
  const deleteTodo = async (todo) => {
    try {
      const { data, error } = await supabase
        .from("todos")
        .delete()
        .eq("id", todo.id);
      await getAllTodos();
    } catch (err) {
      console.log(err);
    }
  };
  const handleKeyPress = (event) => {
    if (event.key === "Enter" && newTask !== "") {
      addNewTodo();
    }
  };
  const logOut = async () => {
    let { error } = await supabase.auth.signOut();
    $user = false;
    goto("/login");
  };
  $: console.log($user);
</script>

<Navbar />
<div class="todos">
  <h4 class="m-5">Welcome {$user ? $user.email : ""}!</h4>
  <div class="add-todo mb-5">
    <input type="text" bind:value={newTask} />
    <button on:click={() => addNewTodo()}>Add Task</button>
  </div>

  {#each todos as todo}
    <Todo {todo} {updateTodo} {deleteTodo} />
  {:else}
    <p>No todos found</p>
  {/each}

  {#if $user}
    <p on:click={logOut} class="switch">Logout</p>
  {:else}
    <button on:click={() => goto("/login")} class="">Login</button>
  {/if}
</div>
<svelte:window on:keypress={handleKeyPress} />

<style>
  .todos{
    display:flex;
    flex-direction: column;
    place-items: center;
  }
  .add-todo {
    display: flex;
    margin-bottom: 0.5em;
  }
  :global(.switch) {
    color: lightskyblue;
    cursor: pointer;
  }
  :global(.switch:hover) {
    text-decoration: underline;
  }
</style>
