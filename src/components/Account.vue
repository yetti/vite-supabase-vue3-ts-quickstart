<template>
  <div class="form-widget">
    <Avatar :url="avatar_url" :size="150" @onUpload="handleImageUpload" />
    <div>
      <label for="email">Email</label>
      <input id="email" type="text" :value="session.user.email" disabled />
    </div>
    <div>
      <label for="username">Username</label>
      <input id="username" v-model="username" type="text" />
    </div>
    <div>
      <label for="website">Website</label>
      <input id="website" v-model="website" type="text" />
    </div>

    <div>
      <button
        class="button block primary"
        :disabled="loading"
        @click.prevent="updateProfile({ username, website, avatar_url })"
      >
        {{ loading ? 'Loading...' : 'Update' }}
      </button>
    </div>
    <div>
      <button class="button block" @click="supabase.auth.signOut()">
        Sign Out
      </button>
    </div>
  </div>
</template>

<script lang="ts">
  import { defineComponent, ref } from '@vue/runtime-core'
  import { supabase } from '/@/services/supabaseClient'
  import { User as AuthUser, Session as AuthSession } from '@supabase/gotrue-js'
  import Avatar from '/@/components/Avatar.vue'

  export default defineComponent({
    name: 'Account',
    components: {
      Avatar,
    },
    // eslint-disable-next-line vue/require-prop-types
    props: ['session'],
    setup(props) {
      const loading = ref(false)
      const username = ref('')
      const website = ref('')
      const avatar_url = ref('')

      const getProfile = async (session: AuthSession) => {
        try {
          loading.value = true
          const user: AuthUser | null = session.user

          let { data, error, status } = await supabase
            .from('profiles')
            .select(`username, website, avatar_url`)
            .eq(`id`, user?.id)
            .single()

          if (error && status !== 400) {
            throw error
          }

          if (data) {
            username.value = data.username
            website.value = data.website
            avatar_url.value = data.avatar_url
          }
        } catch (error) {
          alert(error.message)
        } finally {
          loading.value = false
        }
      }

      const updateProfile = async ({ username, website, avatar_url }) => {
        try {
          loading.value = true
          const user = supabase.auth.user()

          const updates = {
            id: user?.id,
            username,
            website,
            avatar_url,
            updated_at: new Date(),
          }

          let { error } = await supabase.from('profiles').upsert(updates, {
            returning: 'minimal',
          })

          if (error) {
            throw error
          }
        } catch (error) {
          alert(error.message)
        } finally {
          loading.value = false
        }
      }

      const handleImageUpload = async (path: string) => {
        avatar_url.value = path
        await updateProfile({ username, website, avatar_url: path })
      }

      getProfile(props.session)

      return {
        loading,
        username,
        website,
        avatar_url,
        supabase,
        getProfile,
        updateProfile,
        handleImageUpload,
      }
    },
  })
</script>
