<template>
  <div v-if="!username">
    <h1>請輸入您的名稱</h1>
    <input type="text" v-model="newUsername" @keyup.enter="setUsername" />
    <button @click="setUsername">加入聊天室</button>
  </div>
  <ChatRoom v-else :username="username" />
</template>

<script setup>
import { ref } from 'vue';
import ChatRoom from './components/ChatRoom.vue';

const username = ref('');
const newUsername = ref('');

const setUsername = () => {
  if (newUsername.value.trim()) {
    username.value = newUsername.value.trim();
  }
};
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  margin-top: 60px;
}

input[type="text"] {
  padding: 10px;
  margin-right: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #4CAF50;
  color: white;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}
</style>