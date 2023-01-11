<template>
  <form class="form-widget" @submit.prevent="updateProfile">
    <div>
      <label for="email">Email</label>
      <input id="email" type="text" :value="store.user.email" disabled />
    </div>
    <div>
      <label for="username">名称</label>
      <input id="username" type="text" v-model="username" />
    </div>
    <div>
      <label for="website">网址</label>
      <input id="website" type="website" v-model="website" />
    </div>
    <Avatar v-model:path="avatar_url" @upload="updateProfile" />

    <div>
      <input
        type="submit"
        class="button block primary"
        :value="loading ? 'Loading ...' : '修改'"
        :disabled="loading"
      />
    </div>

    <div>
      <button class="button block" @click="signOut" :disabled="loading">
        登出
      </button>
    </div>
  </form>
</template>

<script>
import { supabase } from "../supabase"
import { store } from "../store"
import { onMounted, ref } from "vue"
import Avatar from './Avatar.vue'

export default {
components: {
    Avatar,
  },
  setup() {
    const loading = ref(true)
    const username = ref("")
    const website = ref("")
    const avatar_url = ref("")
    async function getProfile() {
      try {
        loading.value = true
        const { data } = await supabase.auth.getUser()
        store.user = data?.user
        let prfireData = await supabase
          .from("profiles")
          .select(`username, website, avatar_url`)
          .eq("id", store.user.id)
          .single()
        if (prfireData.data.error && prfireData.data.status !== 406) throw error
        if (prfireData.data.data) {
          username.value = prfireData.data.data.username
          website.value = prfireData.data.data.website
          avatar_url.value = prfireData.data.data.avatar_url
        }
      } catch (error) {
        alert(error.message)
      } finally {
        loading.value = false
      }
    }

    async function updateProfile() {
      try {
        loading.value = true
        const { data } = await supabase.auth.getUser()
        store.user = data?.user

        const updates = {
          id: store.user.id,
          username: username.value,
          website: website.value,
          avatar_url: avatar_url.value,
          updated_at: new Date(),
        }

        let { error } = await supabase.from("profiles").upsert(updates, {
          returning: "minimal", // Don't return the value after inserting
        })
        alert('修改成功！')
        if (error) throw error
      } catch (error) {
        alert(error.message)
      } finally {
        loading.value = false
      }
    }

    async function signOut() {
      try {
        loading.value = true
        let { error } = await supabase.auth.signOut()
        if (error) throw error
      } catch (error) {
        alert(error.message)
      } finally {
        loading.value = false
      }
    }

    onMounted(() => {
      getProfile()
    })

    return {
      store,
      loading,
      username,
      website,
      avatar_url,

      updateProfile,
      signOut,
    }
  },
}
</script>
