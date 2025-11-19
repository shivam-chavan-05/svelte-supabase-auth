<script lang="ts">
  import { supabase } from '$lib/supabaseClient';
  import { goto } from '$app/navigation';

  let username = '';
  let email = '';
  let mobile = '';
  let password = '';
  let confirmPassword = '';
  let message = '';
  let messageType: 'success' | 'error' = 'error';
  let loading = false;

  // Validate inputs
  function validate() {
    if (!username.trim()) return 'Please enter your name.';
    if (!email.trim()) return 'Please enter your email.';
    if (!mobile.trim()) return 'Please enter your mobile number.';
    if (!/^\d{10}$/.test(mobile)) return 'Please enter a valid 10-digit mobile number.';
    if (password.length < 6) return 'Password must be at least 6 characters.';
    if (password !== confirmPassword) return 'Passwords do not match.';
    return null;
  }

  // Handle signup
  async function handleSubmit(e: Event) {
    e.preventDefault();
    const err = validate();
    if (err) { message = err; messageType = 'error'; return; }

    loading = true;
    message = '';

    try {
      const { data: exists, error: checkError } = await supabase.rpc('check_user_exists', {
        check_email: email,
        check_phone: mobile
      });

      if (checkError) throw checkError;

      if (exists?.email_exists) { message = 'An account with this email already exists.'; messageType = 'error'; loading = false; return; }
      if (exists?.phone_exists) { message = 'This phone number is already registered.'; messageType = 'error'; loading = false; return; }

      const { data, error: signUpError } = await supabase.auth.signUp({
        email,
        password,
        phone: '+1' + mobile,  // top-level phone column for dashboard visibility
        options: {
          emailRedirectTo: window.location.origin,
          data: {
            username,
            full_name: username,
            display_name: username,
            phone: mobile  // also stored in user_metadata
          }
        }
      });

      loading = false;

      if (signUpError) {
        const em = (signUpError.message || '').toLowerCase();
        if (em.includes('phone')) message = 'This phone number is already registered.';
        else if (em.includes('email')) message = 'An account with this email already exists.';
        else message = signUpError.message;
        messageType = 'error';
        return;
      }

      message = 'Signup successful! Please check your email to confirm.';
      messageType = 'success';
      setTimeout(() => goto('/login'), 1500);

    } catch (err) {
      console.error(err);
      message = 'Something went wrong. Please try again.';
      messageType = 'error';
      loading = false;
    }
  }

  function dismissMessage() { message=''; messageType=''; }
</script>

<main class="w-full max-w-md mx-auto p-6 bg-white rounded-2xl shadow-md mt-12">
  <h2 class="text-2xl font-bold text-center mb-6">Create an account</h2>

  <form on:submit|preventDefault={handleSubmit} class="flex flex-col space-y-4" novalidate>
    <input type="text" placeholder="Full name" bind:value={username} required />
    <input type="email" placeholder="Email" bind:value={email} required />
    <input type="tel" placeholder="Mobile number (10 digits)" bind:value={mobile} maxlength="10" required
           on:input={(e)=> { e.target.value = e.target.value.replace(/\D/g,'').slice(0,10); mobile = e.target.value; }} />
    <input type="password" placeholder="Password" bind:value={password} required />
    <input type="password" placeholder="Confirm password" bind:value={confirmPassword} required />
    <button type="submit" disabled={loading}>{loading ? 'Creating account...' : 'Sign Up'}</button>
  </form>

  <p class="text-center text-sm mt-4">
    Already have an account?
    <button on:click={() => goto('/login')} class="text-blue-600 underline">Sign in</button>
  </p>

  {#if message}
    <div class={`mt-4 p-3 rounded-md text-sm flex items-start justify-between
                ${messageType === 'success' ? 'bg-green-50 border border-green-200 text-green-800' :
                                              'bg-red-50 border border-red-200 text-red-800'}`}>
      <div>{message}</div>
      <button on:click={dismissMessage} class="ml-4 text-sm underline opacity-80">Dismiss</button>
    </div>
  {/if}
</main>

<style>
  input { display:block; margin:8px 0 12px; padding:8px; width:100%; box-sizing:border-box; }
  button { padding:8px 12px; }
</style>
