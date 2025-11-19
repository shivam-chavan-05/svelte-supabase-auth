<!-- Login Page // -->
<script>
  import { onMount, onDestroy } from 'svelte';
  import { supabase } from '$lib/supabaseClient';
  import { goto } from '$app/navigation';

  let email = '';
  let password = '';
  let message = '';
  let messageType = '';
  let loading = false;
  let isForgotPassword = false;

  let user = null;
  let authSub = null;

  onMount(async () => {
    const { data } = await supabase.auth.getSession();
    user = data.session?.user ?? null;
    const { data: sub } = supabase.auth.onAuthStateChange((_event, session) => {
      user = session?.user ?? null;
    });
    authSub = sub;

    try {
      const st = window.history.state && window.history.state.usr;
      if (st && st.unverified) {
        message = 'Please verify your email address before signing in. Check your inbox for the confirmation link.';
        messageType = 'error';
        window.history.replaceState({}, '');
      }
    } catch (e) {}
  });

  onDestroy(() => {
    if (authSub?.unsubscribe) authSub.unsubscribe();
  });

  async function handleSignIn(e) {
    e.preventDefault();
    loading = true;
    message = '';
    messageType = '';
    const { error } = await supabase.auth.signInWithPassword({ email, password });
    loading = false;
    if (error) {
      message = error.message;
      messageType = 'error';
      return;
    }

    const { data: userData, error: userError } = await supabase.auth.getUser();
    if (userError) {
      message = 'Signed in, but unable to verify email status. Please try again.';
      messageType = 'error';
      return;
    }

    const u = userData?.user;
    const isVerified = !!(u?.email_confirmed_at || u?.confirmed_at || u?.email_confirmed);
    if (!isVerified) {
      await supabase.auth.signOut();
      message = 'Please verify your email address before signing in. Check your inbox for the confirmation link.';
      messageType = 'error';
      return;
    }

    goto('/summary');
  }

  async function handleForgotPassword(e) {
    e.preventDefault();
    if (!email) { message = 'Please enter your email address'; messageType = 'error'; return; }
    loading = true;
    const { error } = await supabase.auth.resetPasswordForEmail(email, {
      redirectTo: window.location.origin + '/reset-password'
    });
    loading = false;
    if (error) { message = error.message; messageType = 'error'; } 
    else { message = 'Password reset instructions sent to your email!'; messageType = 'success'; isForgotPassword = false; }
  }

  function dismissMessage() { message = ''; messageType = ''; }
</script>

<main class="w-full max-w-sm mx-auto p-6 bg-white rounded-2xl shadow-md">
  <h2 class="text-2xl font-bold text-center mb-6">{isForgotPassword ? 'Reset Password' : 'Login'}</h2>

  {#if user}
    <section>
      <p>Signed in as <strong>{user.email}</strong></p>
      <div style="margin-top:8px;">
        <button on:click={async () => { await supabase.auth.signOut(); user = null; }} class="btn-primary">Sign out</button>
      </div>
    </section>
  {/if}

  {#if isForgotPassword}
    <form on:submit|preventDefault={handleForgotPassword} class="flex flex-col space-y-4">
      <input type="email" placeholder="Email" bind:value={email} required />
      <div class="flex gap-2">
        <button type="submit" disabled={loading}>{loading ? 'Sending...' : 'Send Reset Link'}</button>
        <button type="button" on:click={() => isForgotPassword = false}>Back to Login</button>
      </div>
    </form>
  {:else}
    <form on:submit|preventDefault={handleSignIn} class="flex flex-col space-y-4">
      <input type="email" placeholder="Email" bind:value={email} required />
      <input type="password" placeholder="Password" bind:value={password} required />
      <button type="submit" disabled={loading}>{loading ? 'Loading...' : 'Sign In'}</button>
      <div class="flex justify-between items-center text-sm">
        <button type="button" on:click={() => isForgotPassword = true}>Forgot Password?</button>
        <button type="button" on:click={() => goto('/signup')}>Sign up</button>
      </div>
    </form>
  {/if}

  {#if message}
    <div style="margin-top:1rem;padding:0.75rem;border-radius:6px;border:1px solid #e5e7eb;">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div>{message}</div>
        <button on:click={dismissMessage} style="text-decoration:underline;">Dismiss</button>
      </div>
    </div>
  {/if}
</main>

<style>
  input { display:block; margin:8px 0 12px; padding:8px; width:100%; box-sizing:border-box; }
  button { padding:8px 12px; }
</style>
