<template>
  <div>
    <input v-model="errorCode" placeholder="Masukkan kode error ATM" />
    <button @click="getSolution" :disabled="loading">
      {{ loading ? 'Mencari...' : 'Cari Solusi' }}
    </button>
    <div v-if="solution">
      <h3>Jawaban Asisten AI:</h3>
      <p style="white-space: pre-wrap;">{{ solution }}</p>
    </div>
     <div v-if="error">
      <p style="color: red;">{{ error }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const errorCode = ref('');
const solution = ref('');
const loading = ref(false);
const error = ref('');

const getSolution = async () => {
  if (!errorCode.value) return;
  loading.value = true;
  solution.value = '';
  error.value = '';
  try {
    // Ganti URL dengan URL backend Anda yang sudah di-deploy di Render/Railway
    const response = await axios.post('https://api-atm-anda.onrender.com/ask', {
      errorCode: errorCode.value,
    });
    solution.value = response.data.answer;
  } catch (err) {
    error.value = err.response?.data?.error || 'Gagal terhubung ke server.';
  } finally {
    loading.value = false;
  }
};
</script>