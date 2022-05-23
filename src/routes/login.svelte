<script>
  import { user } from '$lib/sessionStore';
  import Auth from "$lib/Auth.svelte";
  import Profile from "$lib/Profile.svelte";
  import { goto } from "$app/navigation";
  import Navbar from "$lib/Navbar.svelte";

</script>

<Navbar />
<div class="container" style="padding: 50px 0 100px 0;">
    {#if $user}
        <Profile />
    {:else}
        <Auth />
    {/if}
</div>

    <button on:click={() => goto("/")} class="">Home</button>
