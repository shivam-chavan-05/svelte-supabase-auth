<script>
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';
  import { goto } from '$app/navigation';

  let user = null;
  let loading = true;

  onMount(async () => {
    const { data } = await supabase.auth.getSession();
    user = data.session?.user ?? null;
    loading = false;
    if (!user) {
      // Not logged in -> redirect to login
      goto('/login');
    }
  });

  async function signOut() {
    await supabase.auth.signOut();
    goto('/login');
  }
</script>

{#if loading}
  <p>Loading...</p>
{:else}
  {#if user}
    <main class="p-6">
      <h1 class="text-2xl font-bold">Summary / Dashboard</h1>
      <p>Welcome, <strong>{user.user_metadata?.display_name ?? user.email}</strong></p>
      <p style="margin-top:1rem">Your user ID: {user.id}</p>
      <div style="margin-top:1rem">
        <button on:click={signOut}>Sign out</button>
      </div>
    </main>
  {:else}
    <p>Redirecting to login...</p>
  {/if}
{/if}
