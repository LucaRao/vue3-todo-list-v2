<template>
  <form class="row flex flex-center">
    <div class="col-6 form-widget">
      <h1 class="header">Memfiredb + Vue 3</h1>
      <p class="description">使用下面的电子邮件通过魔术链接登录</p>
      <div>
        <input
          class="inputField"
          type="email"
          placeholder="邮箱"
          v-model="email"
        />
      </div>
      <div>
        <input
          class="inputField"
          type="password"
          placeholder="密码"
          v-model="password"
        />
      </div>
      <div>
        <input
        @click="singup"
          class="button block"
          :value="loading ? 'Loading' : '注册'"
          :disabled="loading"
        />
      </div>
      <div>
        <input
        @click="handleLogin"
          class="button block"
          :value="loading ? 'Loading' : '登录'"
          :disabled="loading"
        />
      </div>
    </div>
  </form>
</template>

<script>
import { ref } from "vue";
import { supabase } from "../supabase";

export default {
  setup() {
    const loading = ref(false);
    const email = ref("");
    const password = ref("");

    const handleLogin = async () => {
      try {
        loading.value = true;
        const { error } = await supabase.auth.signInWithPassword({ email: email.value,password:password.value });
        if (error) throw error;
      } catch (error) {
        alert(error.error_description || error.message);
      } finally {
        loading.value = false;
      }
    };
    const singup = async ()=>{
      try {
        loading.value = true;
        const { error } = await supabase.auth.signUp({ email: email.value,password:password.value });
        if (error) throw error;
      } catch (error) {
        alert(error.error_description || error.message);
      } finally {
        loading.value = false;
      }
    }

    return {
      loading,
      email,
      password,
      handleLogin,
      singup
    };
  },
};
</script>