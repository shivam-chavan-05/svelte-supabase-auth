<!-- Reset Password Page // -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';
  import { goto } from '$app/navigation';

  let newPassword = '';
  let confirmNewPassword = '';
  let message = '';
  let messageType: 'success' | 'error' = 'error';
  let loading = false;

  onMount(() => {
  });

  async function handlePasswordReset(e: Event) {
    e.preventDefault();
    message = '';
    messageType = 'error';

    // Validate passwords
    if (newPassword !== confirmNewPassword) {
      message = 'Passwords do not match';
      return;
    }
    if (newPassword.length < 6) {
      message = 'Password must be at least 6 characters';
      return;
    }

    loading = true;

    // Update password
    const { error } = await supabase.auth.updateUser({ password: newPassword });
    if (error) {
      message = error.message || 'Unable to update password. The reset link might be expired.';
      loading = false;
      return;
    }

    await supabase.auth.signOut();

    loading = false;
    message = 'Password updated successfully! Redirecting to login...';
    messageType = 'success';

    // Redirect after short delay
    setTimeout(() => goto('/login'), 1600);
  }
</script>

<main class="w-full max-w-sm mx-auto p-6 bg-white rounded-2xl shadow-md mt-12">
  <h2 class="text-2xl font-bold text-center mb-6">Reset Password</h2>

  <form on:submit|preventDefault={handlePasswordReset} class="flex flex-col space-y-4">
    <input type="password" placeholder="New password" bind:value={newPassword} required />
    <input type="password" placeholder="Confirm new password" bind:value={confirmNewPassword} required />
    <button type="submit" disabled={loading}>
      {loading ? 'Saving...' : 'Save new password'}
    </button>
  </form>

  {#if message}
    <div class={`mt-4 p-3 rounded-md text-sm flex items-start justify-between
                ${messageType === 'success' ? 'bg-green-50 border border-green-200 text-green-800' :
                                              'bg-red-50 border border-red-200 text-red-800'}`}>
      <div>{message}</div>
    </div>
  {/if}
</main>

<style>
  input { display:block; margin:8px 0 12px; padding:8px; width:100%; box-sizing:border-box; }
  button { padding:8px 12px; }
</style>
