;
<template>
  <div>
    <template v-if="avatarUrl">
      <img
        :src="avatarUrl"
        alt="Avatar"
        class="avatar image"
        :style="{ height: size, width: size }"
      />
    </template>
    <template v-else>
      <div class="avatar no-image" :style="{ height: size, width: size }" />
    </template>
  </div>
  <div :style="{ width: size }">
    <label class="button primary block" for="single">{{
      uploading ? 'Uploading...' : 'Upload'
    }}</label>
    <input
      id="single"
      :style="{ width: size }"
      type="file"
      accept="image/*"
      :disabled="uploading"
      @change="uploadAvatar"
    />
  </div>
</template>

<script lang="ts">
  import { defineComponent, ref, watch } from '@vue/runtime-core'
  import { supabase } from '/@/services/supabaseClient'

  export default defineComponent({
    name: 'Avatar',
    props: {
      url: {
        default: '',
        type: String,
      },
      size: {
        default: 150,
        type: Number,
      },
    },
    emits: ['onUpload'],
    setup(props, ctx) {
      const avatarUrl = ref('')
      const uploading = ref(false)

      watch(
        () => props?.url,
        (newValue, _oldValue) => {
          downloadImage(newValue)
        }
      )

      const downloadImage = async (path: string) => {
        console.log('download path', path)

        if (!path) {
          avatarUrl.value = ''
          return
        }

        const { data, error } = await supabase.storage
          .from('avatars')
          .download(path)

        if (error) {
          throw error
        }

        avatarUrl.value = URL.createObjectURL(data)
      }

      const uploadAvatar = async (event: InputEvent) => {
        try {
          uploading.value = true

          const target = <HTMLInputElement>event.target

          if (!target.files || target?.files.length == 0) {
            throw new Error('You must select an image to upload.')
          }

          const file = target?.files[0]
          const fileExt = file.name.split('.').pop()
          const fileName = `${Math.random()}.${fileExt}`
          const filePath = `${fileName}`

          let { error: uploadError } = await supabase.storage
            .from('avatars')
            .upload(filePath, file)

          if (uploadError) {
            throw uploadError
          }

          ctx.emit('onUpload', filePath)
        } catch (error) {
          alert(error.message)
        } finally {
          uploading.value = false
        }
      }

      return {
        avatarUrl,
        uploading,
        uploadAvatar,
      }
    },
  })
</script>
