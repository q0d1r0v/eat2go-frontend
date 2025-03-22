<script setup>
import { onMounted, ref } from "vue";
import axios from "axios";

let tg = ref(null);
const user = ref(null);

async function sendDataToBackend() {
  try {
    const response = await axios.post("https://ac46-213-230-120-88.ngrok-free.app/auth/telegram/data", {
      user: user.value, // user?.value o‘rniga user.value
      message: "Salom, backend!",
    });

    console.log("Server javobi:", response.data);
  } catch (error) {
    console.error("Xatolik:", error.response ? error.response.data : error.message);
  }
}

onMounted(() => {
  if (typeof window.Telegram !== "undefined") {
    tg.value = window.Telegram.WebApp;
    tg.value.expand(); // Fullscreen qilish
    user.value = tg.value.initDataUnsafe?.user || null; // Telegram user ma'lumotlarini olish

    console.log("Telegram WebApp yuklandi:", tg.value);
    console.log("Foydalanuvchi ma’lumotlari:", user.value);
  } else {
    console.error("Telegram WebApp yuklanmadi! Iltimos, Telegram ichida oching.");
  }
});
</script>

<template>
  <div style="background: white;">
    <h1>Telegram Mini App</h1>
    <p v-if="user">Foydalanuvchi: {{ user.first_name }}</p>
    <button @click="sendDataToBackend()">NestJS ga so‘rov yuborish</button>
  </div>
</template>
