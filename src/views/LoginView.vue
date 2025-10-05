<script setup lang="ts">
import { ref } from 'vue'
import api from '@/lib/api'
import type { Credentials } from '@/types'
import type { AxiosError } from 'axios'

const username = ref('')
const password = ref('')

async function login() {
  console.log('login')
  if (username.value === '' || password.value === '') {
    alert('Please fill in all fields')
    return
  }
  try {
    const credentials: Credentials = {
      username: username.value,
      password: password.value,
    }
    username.value = ''
    password.value = ''
    const res = await api.post('/auth/login', credentials)
    localStorage.setItem('token', res.data.token)
    // TODO: localStorage.removeItem("token");
    window.location.href = '/'
  } catch (err) {
    const error = err as AxiosError<{ error: string }> | Error
    if ('response' in error && error.response?.data?.error) {
      alert(error.response.data.error)
    } else {
      alert(error.message)
    }
  } finally {
    //
  }
}
</script>

<template lang="pug">
    main
        form.login-form(@submit.prevent="login")
            h1 Log in
            label Username
            input(type="text" required v-model="username")
            label Password
            input(type="password" required v-model="password")
            button(@click="login") Log in
</template>

<style lang="scss" scoped>
.login-form {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  width: 100%;
  max-width: 320px;
  margin: 0 auto;

  h1 {
    text-align: center;
  }

  label {
    font-size: 0.75rem;
  }

  input {
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 0.25rem;
  }

  button {
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 0.25rem;
    background-color: #ccc;
    cursor: pointer;
  }
}
</style>
