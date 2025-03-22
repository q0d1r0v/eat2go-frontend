<script setup>
import { onMounted, ref } from "vue";
import axios from "axios";

const user = ref(null);
const isLoading = ref(true);
const error = ref(null);
const debug = ref({
  hasWindow: false,
  hasTelegram: false,
  hasWebApp: false,
  hasInitData: false,
  rawInitData: null
});

// Backend API URL
const API_URL = "https://5da0-213-230-120-88.ngrok-free.app/auth/telegram/data";

async function sendDataToBackend() {
  try {
    if (!user.value) {
      throw new Error("Foydalanuvchi ma'lumotlari mavjud emas");
    }

    const response = await axios.post(API_URL, {
      user: user.value,
      message: "Salom, backend!",
    });

    console.log("Server javobi:", response.data);
    alert("Ma'lumotlar muvaffaqiyatli yuborildi!");
  } catch (error) {
    console.error("Xatolik:", error.response ? error.response.data : error.message);
    alert(`Xatolik yuz berdi: ${error.response?.data?.message || error.message}`);
  }
}

function initTelegramWebApp() {
  isLoading.value = true;
  error.value = null;

  try {
    // Debug checks
    debug.value.hasWindow = typeof window !== 'undefined';
    debug.value.hasTelegram = debug.value.hasWindow && !!window.Telegram;
    debug.value.hasWebApp = debug.value.hasTelegram && !!window.Telegram.WebApp;
    debug.value.hasInitData = debug.value.hasWebApp && !!window.Telegram.WebApp.initData;

    if (debug.value.hasWebApp) {
      // Capture raw data for debugging
      debug.value.rawInitData = {
        initData: window.Telegram.WebApp.initData,
        initDataUnsafe: window.Telegram.WebApp.initDataUnsafe
      };

      // Expand the app
      window.Telegram.WebApp.expand();

      // Try multiple approaches to get user data
      if (window.Telegram.WebApp.initDataUnsafe?.user) {
        // Standard approach
        user.value = window.Telegram.WebApp.initDataUnsafe.user;
      } else if (window.Telegram.WebApp.initData) {
        // Try to parse initData manually
        try {
          const params = new URLSearchParams(window.Telegram.WebApp.initData);
          const userParam = params.get('user');
          if (userParam) {
            user.value = JSON.parse(userParam);
          }
        } catch (e) {
          console.error("Failed to parse initData:", e);
        }
      }

      console.log("Telegram WebApp yuklandi");
      console.log("Foydalanuvchi ma'lumotlari:", user.value);
    } else {
      throw new Error("Telegram WebApp yuklanmadi! Iltimos, Telegram ichida oching.");
    }
  } catch (err) {
    error.value = err.message;
    console.error(err.message);
  } finally {
    isLoading.value = false;
  }
}

onMounted(() => {
  // Try immediately on mount
  initTelegramWebApp();

  // If not successful, try again after a delay
  if (!user.value) {
    setTimeout(() => {
      initTelegramWebApp();
    }, 1000);
  }
});
</script>

<template>
  <div class="telegram-mini-app">
    <!-- Loading state -->
    <div v-if="isLoading" class="loading-container">
      <div class="loading-spinner"></div>
      <p>Yuklanmoqda...</p>
    </div>

    <!-- Error state -->
    <div v-else-if="error" class="error-container">
      <h2>Xatolik yuz berdi</h2>
      <p>{{ error }}</p>
      <button @click="initTelegramWebApp" class="retry-button">Qayta urinish</button>
    </div>

    <!-- Success state with user -->
    <div v-else-if="user" class="content-container">
      <h1>Telegram Mini App</h1>

      <div class="user-info">
        <h2>Foydalanuvchi ma'lumotlari</h2>
        <p><strong>Ism:</strong> {{ user.first_name }}</p>
        <p v-if="user.last_name"><strong>Familiya:</strong> {{ user.last_name }}</p>
        <p v-if="user.username"><strong>Username:</strong> @{{ user.username }}</p>
        <p v-if="user.id"><strong>Telegram ID:</strong> {{ user.id }}</p>
      </div>

      <div class="action-buttons">
        <button @click="sendDataToBackend" class="primary-button">
          NestJS ga so'rov yuborish
        </button>
      </div>
    </div>

    <!-- No user data but app loaded -->
    <div v-else class="content-container">
      <h1>Telegram Mini App</h1>

      <div class="no-user-data">
        <h2>Foydalanuvchi ma'lumotlari mavjud emas</h2>
        <p>Iltimos, mini-appni Telegram orqali oching.</p>
      </div>

      <!-- Debug info -->
      <div class="debug-info">
        <h3>Debugging ma'lumotlari:</h3>
        <ul>
          <li>Window mavjud: {{ debug.hasWindow ? '✅' : '❌' }}</li>
          <li>Telegram mavjud: {{ debug.hasTelegram ? '✅' : '❌' }}</li>
          <li>WebApp mavjud: {{ debug.hasWebApp ? '✅' : '❌' }}</li>
          <li>initData mavjud: {{ debug.hasInitData ? '✅' : '❌' }}</li>
        </ul>
        <div v-if="debug.rawInitData">
          <h4>Raw InitData:</h4>
          <pre>{{ JSON.stringify(debug.rawInitData, null, 2) }}</pre>
        </div>
      </div>

      <div class="action-buttons">
        <button @click="initTelegramWebApp" class="retry-button">
          Qayta urinish
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.telegram-mini-app {
  background-color: white;
  color: #222;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  max-width: 600px;
  margin: 0 auto;
  padding: 16px;
  min-height: 100vh;
}

h1 {
  font-size: 24px;
  margin-bottom: 24px;
  text-align: center;
}

h2 {
  font-size: 18px;
  margin-bottom: 16px;
}

.loading-container,
.error-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 60vh;
  text-align: center;
}

.loading-spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin-bottom: 16px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

.user-info,
.no-user-data,
.debug-info {
  background-color: #f5f5f5;
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 24px;
}

.debug-info {
  background-color: #fff8e1;
  border: 1px solid #ffd54f;
  font-size: 14px;
  overflow-x: auto;
}

.debug-info pre {
  white-space: pre-wrap;
  background-color: #f5f5f5;
  padding: 8px;
  border-radius: 4px;
  font-size: 12px;
}

.action-buttons {
  display: flex;
  justify-content: center;
  margin-top: 24px;
}

.primary-button,
.retry-button {
  background-color: #40a7e3;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 12px 20px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.primary-button:hover,
.retry-button:hover {
  background-color: #3498db;
}

.retry-button {
  background-color: #e74c3c;
}

.retry-button:hover {
  background-color: #c0392b;
}

.error-container {
  color: #e74c3c;
}

.no-user-data {
  text-align: center;
  padding: 24px;
  background-color: #f8f9fa;
  border-radius: 8px;
}
</style>