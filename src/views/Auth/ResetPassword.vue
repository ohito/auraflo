<template>
  <div class="min-h-screen flex items-center justify-center bg-slate-50 p-6">
    <div class="w-full max-w-md bg-white rounded-2xl shadow-lg border border-gray-100 overflow-hidden">
      <!-- header -->
      <div class="px-6 py-5 border-b border-gray-100">
        <h1 class="text-xl font-semibold text-slate-800">
          {{ mode === 'request' ? 'Reset Password' : 'Set a New Password' }}
        </h1>
        <p class="text-sm text-slate-500 mt-1">
          {{ mode === 'request'
            ? 'Enter your email and we will send instructions to reset your password.'
            : 'Enter a new password for your account.' }}
        </p>
      </div>

      <!-- body -->
      <form @submit.prevent="onSubmit" class="px-6 py-6">
        <!-- request mode (email) -->
        <div v-if="mode === 'request'">
          <label class="block text-sm text-slate-700 mb-2" for="email">Email</label>
          <input
            id="email"
            v-model="email"
            type="email"
            required
            placeholder="you@company.com"
            class="w-full rounded-md border border-gray-200 px-3 py-2 text-sm focus:outline-none focus:ring-1 focus:ring-emerald-400"
          />
          <p v-if="error" class="mt-2 text-xs text-red-600">{{ error }}</p>

          <div class="mt-6">
            <button
              type="submit"
              :disabled="loading"
              class="w-full inline-flex justify-center items-center gap-2 rounded-lg bg-emerald-500 px-4 py-2 text-white text-sm font-medium hover:bg-emerald-600 disabled:opacity-60"
            >
              <svg v-if="!loading" xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 12H8m8 0a4 4 0 11-8 0 4 4 0 018 0z" />
              </svg>
              <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 animate-spin" viewBox="0 0 24 24" fill="none">
                <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" stroke-opacity="0.25" />
                <path d="M22 12a10 10 0 00-10-10" stroke="currentColor" stroke-width="4" stroke-linecap="round"/>
              </svg>
              {{ loading ? 'Sending...' : 'Send reset link' }}
            </button>
          </div>

          <div class="mt-4 text-center text-sm">
            <a href="/login" class="text-emerald-600 hover:underline">Back to sign in</a>
          </div>
        </div>

        <!-- reset mode (new password) -->
        <div v-else>
          <input type="hidden" :value="token" />

          <label class="block text-sm text-slate-700 mb-2" for="password">New password</label>
          <div class="relative">
            <input
              id="password"
              :type="showPassword ? 'text' : 'password'"
              v-model="password"
              minlength="8"
              required
              placeholder="Create a strong password"
              class="w-full rounded-md border border-gray-200 px-3 py-2 text-sm focus:outline-none focus:ring-1 focus:ring-emerald-400 pr-10"
            />
            <button type="button" @click="showPassword = !showPassword" class="absolute right-2 top-1/2 -translate-y-1/2 text-slate-400">
              <span v-if="showPassword" aria-hidden>🙈</span>
              <span v-else aria-hidden>👁️</span>
            </button>
          </div>

          <label class="block text-sm text-slate-700 mt-4 mb-2" for="confirm">Confirm password</label>
          <input
            id="confirm"
            :type="showPassword ? 'text' : 'password'"
            v-model="confirm"
            minlength="8"
            required
            placeholder="Repeat the password"
            class="w-full rounded-md border border-gray-200 px-3 py-2 text-sm focus:outline-none focus:ring-1 focus:ring-emerald-400"
          />

          <p v-if="error" class="mt-2 text-xs text-red-600">{{ error }}</p>
          <p v-if="success" class="mt-2 text-xs text-emerald-600">{{ success }}</p>

          <div class="mt-6">
            <button
              type="submit"
              :disabled="loading"
              class="w-full inline-flex justify-center items-center gap-2 rounded-lg bg-emerald-500 px-4 py-2 text-white text-sm font-medium hover:bg-emerald-600 disabled:opacity-60"
            >
              <svg v-if="!loading" xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 12h14M12 5l7 7-7 7" />
              </svg>
              <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 animate-spin" viewBox="0 0 24 24" fill="none">
                <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" stroke-opacity="0.25" />
                <path d="M22 12a10 10 0 00-10-10" stroke="currentColor" stroke-width="4" stroke-linecap="round"/>
              </svg>
              {{ loading ? 'Saving...' : 'Set new password' }}
            </button>
          </div>

          <div class="mt-4 text-center text-sm">
            <a href="/login" class="text-emerald-600 hover:underline">Back to sign in</a>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

/**
 * Props:
 *  - mode: 'request' | 'reset' (default 'request')
 *  - token: string (for reset mode, provided by route query param or parent)
 *
 * Emits:
 *  - 'sent' when request link sent
 *  - 'reset' when password successfully set
 *
 * Note: replace commented axios parts with your API calls.
 */

defineProps({
  mode: { type: String, default: 'request' },
  token: { type: String, default: '' }
})
defineEmits(['sent','reset'])

const email = ref('')
const password = ref('')
const confirm = ref('')
const showPassword = ref(false)
const loading = ref(false)
const error = ref('')
const success = ref('')

async function onSubmit() {
  error.value = ''
  success.value = ''
  loading.value = true

  try {
    if (props.mode === 'request' || props.mode === undefined) {
      // --- send reset link to email ---
      // Example (uncomment and set axios):
      // await axios.post('/api/auth/password-reset', { email: email.value })
      await fakeWait(900) // remove in production
      emit('sent', { email: email.value })
      success.value = 'If an account exists with that email, you will receive instructions.'
    } else {
      // --- set new password using token ---
      if (password.value.length < 8) {
        throw new Error('Password must be at least 8 characters.')
      }
      if (password.value !== confirm.value) {
        throw new Error('Passwords do not match.')
      }
      // Example API:
      // await axios.post('/api/auth/password-reset/confirm', { token: props.token, password: password.value })
      await fakeWait(900) // remove in production
      emit('reset')
      success.value = 'Your password has been updated. You can now sign in.'
      password.value = ''
      confirm.value = ''
    }
  } catch (err) {
    error.value = err.message ?? 'Something went wrong. Please try again.'
  } finally {
    loading.value = false
  }
}

// small fake wait helper for demo
function fakeWait(ms=700){ return new Promise(r=>setTimeout(r,ms)) }
</script>

<style scoped>
/* minor responsive tweaks */
@media (max-width: 420px) {
  .max-w-md { max-width: 100% }
}
</style>
