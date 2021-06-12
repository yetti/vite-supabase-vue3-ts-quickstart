<template>
  <div class="row flex flex-center">
    <div class="col-6 form-widget">
      <h1 class="header">Supabase + Vue.js</h1>
      <p class="description">Sign in via magic link with your email below</p>
      <div>
        <input
          v-model="email"
          class="inputField"
          type="email"
          placeholder="Your email"
        />
      </div>
      <div>
        <button
          class="button block"
          :disabled="loading"
          @click.prevent="handleLogin(email)"
        >
          <span>{{ loading ? 'Loading...' : 'Send magic link' }}</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
  import { defineComponent, ref } from '@vue/runtime-core'
  import { supabase } from '/@/services/supabaseClient'

  export default defineComponent({
    name: 'Auth',
    setup() {
      const email = ref('')
      const loading = ref(false)

      const handleLogin = async (email: string) => {
        try {
          loading.value = true
          const { error } = await supabase.auth.signIn({ email })
          if (error) throw error
          alert('Check your email for the login link!')
        } catch (error) {
          alert(error.error_description || error.message)
        } finally {
          loading.value = false
        }
      }

      return {
        email,
        loading,
        handleLogin,
      }
    },
  })
</script>
