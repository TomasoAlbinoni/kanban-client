<script setup lang="ts">
import { ref } from 'vue'
import api from '@/lib/api'
import type { AxiosError } from 'axios'
import type { Credentials } from '@/types'

const username = ref('')
const password = ref('')

async function signup() {
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
    const res = await api.post('/auth/signup', credentials)
    window.location.href = '/login'
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
        form.signup-form(@submit.prevent="signup")
            h1 Signup
            label Username
            input(type="text" required v-model="username")
            label Password
            input(type="password" required v-model="password")
            button(@click="signup") Signup
</template>

<style lang="scss" scoped>
.signup-form {
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
