<!-- <template>
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
    const response = await axios.post('http://localhost:3000/ask', {
      errorCode: errorCode.value,
    });
    solution.value = response.data.answer;
  } catch (err) {
    error.value = err.response?.data?.error || 'Gagal terhubung ke server.';
  } finally {
    loading.value = false;
  }
};
</script> -->

<template>
  <div class="chat-container">
    <div class="message-history" ref="messageContainer">
      <div v-for="message in messages" :key="message.id" class="message-wrapper">
        <div class="message" :class="message.sender === 'user' ? 'user-message' : 'ai-message'">
          <div class="avatar">
            {{ message.sender === 'user' ? '🧑‍💻' : '🤖' }}
          </div>
          <div class="text-content">
            <p>{{ message.text }}</p>
          </div>
        </div>
      </div>
      <div v-if="isLoading" class="message-wrapper">
        <div class="message ai-message">
           <div class="avatar">🤖</div>
           <div class="text-content typing-indicator">
             <span></span><span></span><span></span>
           </div>
        </div>
      </div>
    </div>

    <div class="input-area">
      <form @submit.prevent="sendMessage" class="input-form">
        <input
          v-model="newMessage"
          type="text"
          placeholder="Masukkan kode error di sini..."
          :disabled="isLoading"
        />
        <button type="submit" :disabled="isLoading || !newMessage">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
            <path d="M3.478 2.405a.75.75 0 00-.926.94l2.432 7.905H13.5a.75.75 0 010 1.5H4.984l-2.432 7.905a.75.75 0 00.926.94 60.519 60.519 0 0018.445-8.986.75.75 0 000-1.218A60.517 60.517 0 003.478 2.405z" />
          </svg>
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, nextTick } from 'vue';
import axios from 'axios';

// --- State Management ---
// Menyimpan semua pesan dalam percakapan
const messages = ref([]);
// Menyimpan teks yang sedang diketik pengguna
const newMessage = ref('');
// Menandakan apakah sedang menunggu jawaban dari AI
const isLoading = ref(false);
// Referensi ke elemen DOM untuk auto-scrolling
const messageContainer = ref(null);

// --- API Endpoint ---
// PENTING: Ganti URL ini dengan URL backend Node.js Anda yang di-deploy di Render/Railway
const API_ENDPOINT = 'http://api.personal-assistant.biz.id/ask';

// --- Functions ---
const sendMessage = async () => {
  if (!newMessage.value.trim() || isLoading.value) return;

  const userMessage = {
    id: Date.now(),
    text: newMessage.value,
    sender: 'user',
  };
  messages.value.push(userMessage);
  
  const currentMessageText = newMessage.value;
  newMessage.value = ''; // Kosongkan input setelah dikirim
  isLoading.value = true;

  try {
    const response = await axios.post(API_ENDPOINT, {
      errorCode: currentMessageText,
    });
    
    const aiMessage = {
      id: Date.now() + 1,
      text: response.data.answer,
      sender: 'ai',
    };
    messages.value.push(aiMessage);

  } catch (error) {
    const errorMessage = {
      id: Date.now() + 1,
      text: error.response?.data?.error || 'Maaf, terjadi kesalahan. Coba lagi nanti.',
      sender: 'ai',
    };
    messages.value.push(errorMessage);
  } finally {
    isLoading.value = false;
  }
};

// --- Lifecycle & Effects ---
// Fungsi untuk auto-scroll ke pesan terbaru
const scrollToBottom = async () => {
  await nextTick(); // Tunggu DOM selesai di-update
  if (messageContainer.value) {
    messageContainer.value.scrollTop = messageContainer.value.scrollHeight;
  }
};

// Saat komponen pertama kali dimuat, ambil riwayat chat dari localStorage
onMounted(() => {
  const savedMessages = localStorage.getItem('chat-history');
  if (savedMessages) {
    messages.value = JSON.parse(savedMessages);
  }
  scrollToBottom();
});

// Setiap kali ada pesan baru, simpan ke localStorage dan auto-scroll
watch(messages, (newMessages) => {
  localStorage.setItem('chat-history', JSON.stringify(newMessages));
  scrollToBottom();
}, { deep: true }); // 'deep: true' penting untuk array of objects

</script>

<style scoped>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 90vh; /* Mengisi tinggi layar */
  max-width: 100%;
  margin: 0;
  /* border: 1px solid #e0e0e0; */
  border-radius: 1px;
  overflow: hidden;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  /* background-color: #f9f9f9; */
}

.message-history {
  flex-grow: 1;
  overflow-y: auto;
  padding: 10px;
  display: flex;
  flex-direction: column;
  gap: 15px;
  width: 100%;
}

.message-wrapper {
  display: flex;
  flex-direction: column;
}

.message {
  display: flex;
  gap: 10px;
  max-width: 100%;
}

.avatar {
  font-size: 1.5rem;
  padding-top: 5px;
  padding-right: 15px;
}

.text-content {
  padding: 10px 15px;
  border-radius: 18px;
  white-space: pre-wrap; /* Agar baris baru dan spasi tampil benar */
  line-height: 1.5;
}

.user-message {
  align-self: flex-end;
  flex-direction: row-reverse;
}
.user-message .text-content {
  background-color: #E9EEF6;
  color: black;
  border-bottom-right-radius: 4px;
}

.ai-message {
  align-self: flex-start;
}
.ai-message .text-content {
  /* background-color: #e0e0e0; */
  color: #333;
  border-bottom-left-radius: 4px;
}

.input-area {
  padding: 10px;
  border-top: 1px solid #e0e0e0;
  background-color: #fff;
}

.input-form {
  display: flex;
  gap: 10px;
}

input[type="text"] {
  flex-grow: 1;
  border: 1px solid #ccc;
  border-radius: 20px;
  padding: 10px 15px;
  font-size: 1rem;
}
input[type="text"]:focus {
  outline: none;
  border-color: #0b57d0;
}

button {
  border: none;
  background-color: #0b57d0;
  color: white;
  border-radius: 50%;
  width: 44px;
  height: 44px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.2s;
}
button:hover:not(:disabled) {
  background-color: #0a4cb5;
}
button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

/* Typing indicator animation */
.typing-indicator span {
  height: 8px;
  width: 8px;
  background-color: #888;
  border-radius: 50%;
  display: inline-block;
  animation: bounce 1.4s infinite ease-in-out both;
}
.typing-indicator span:nth-of-type(1) { animation-delay: -0.32s; }
.typing-indicator span:nth-of-type(2) { animation-delay: -0.16s; }
@keyframes bounce {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1.0); }
}
</style>