<template>
    <div class="chat-room-container">
        <div class="header">
            <div class="greeting">你好，{{ username }}！</div>
            <div class="online-users">
                在線用戶：
                <span v-for="(user, index) in onlineUsers" :key="index">
                    {{ user }}<span v-if="index < onlineUsers.length - 1">、</span>
                </span>
            </div>
        </div>
        <div class="chat-dialog">
            <div v-for="(msg, index) in messages" :key="index" class="message"
                :class="{ 'my-message': msg.name === username }">
                <span class="sender" v-if="msg.name && msg.name !== username">{{ msg.name }}: </span>
                <span class="content">{{ msg.content }}</span>
            </div>
        </div>
        <div class="input-area">
            <input type="text" v-model="newMessage" @keyup.enter="sendMessage" placeholder="輸入訊息..." />
            <button @click="sendMessage">發送</button>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const props = defineProps({
    username: String,
});

const messages = ref([]);
const newMessage = ref('');
const onlineUsers = ref([]);
const socket = ref(null);

onMounted(() => {
    // socket.value = new WebSocket('ws://localhost:8080/ws');
    // socket.value = new WebSocket('ws://192.168.251.92:8080/ws');
    // socket.value = new WebSocket('ws://0.0.0.0:8080/ws');
    socket.value = new WebSocket('wss://go-ws-chat.onrender.com/ws');

    socket.value.onopen = () => {
        console.log('WebSocket 連線已開啟');
        socket.value.send(JSON.stringify({ type: "join", name: props.username, content: '' }));
    };

    socket.value.onmessage = (event) => {
        const data = JSON.parse(event.data);
        switch (data.type) {
            case 'join':
                if (data.content.includes('加入了聊天室') && !onlineUsers.value.includes(data.content.split(' ')[0])) {
                    onlineUsers.value.push(data.content.split(' ')[0]);
                }
                messages.value.push({ name: '', content: data.content });
                break;
            case 'leave':
                if (data.content.includes('離開了聊天室')) {
                    onlineUsers.value = onlineUsers.value.filter(user => user !== data.content.split(' ')[0]);
                }
                messages.value.push({ name: '', content: data.content });
                break;
            case 'message':
                messages.value.push(data);
                break;
            case 'online_users':
                onlineUsers.value = data.users;
                break;
            default:
                console.log('未知的消息類型:', data);
        }
    };

    socket.value.onclose = () => {
        console.log('WebSocket 連線已關閉');
    };


});

const sendMessage = () => {
    if (newMessage.value.trim()) {
        socket.value.send(JSON.stringify({ type: "message", name: props.username, content: newMessage.value }));
        newMessage.value = '';
    }
};
</script>


<style scoped>
.chat-room-container {
    display: flex;
    flex-direction: column;
    height: 400px;
    width: 500px;
    border: 1px solid #ccc;
    border-radius: 5px;
    overflow: hidden;
    background-color: #f9f9f9;
}

.header {
    background-color: #eee;
    padding: 10px;
    border-bottom: 1px solid #ddd;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.greeting {
    font-weight: bold;
}

.online-users {
    font-size: 0.9em;
    color: #777;
}

.chat-dialog {
    flex-grow: 1;
    padding: 10px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
}

.message {
    background-color: #fff;
    border: 1px solid #e0e0e0;
    padding: 8px 12px;
    margin-bottom: 8px;
    border-radius: 5px;
    word-break: break-word;
    align-self: flex-start;
    /* 預設靠左對齊 */
}

.my-message {
    background-color: #dcf8c6;
    align-self: flex-end;
    /* 自己的訊息靠右對齊 */
}

.sender {
    font-weight: bold;
    margin-right: 5px;
    color: #333;
}

.content {
    color: #555;
}

.input-area {
    display: flex;
    padding: 10px;
    border-top: 1px solid #ddd;
    background-color: #fff;
    /* 確保輸入區域有背景色 */
}

.input-area input[type="text"] {
    flex-grow: 1;
    padding: 8px;
    border: 1px solid #ccc;
    /* 增加邊框 */
    border-radius: 5px 0 0 5px;
    color: #333;
    /* 確保文字顏色可見 */
    background-color: #fff;
    /* 確保輸入框有背景色 */
}

.input-area button {
    padding: 8px 15px;
    border: none;
    border-radius: 0 5px 5px 0;
    background-color: #007bff;
    color: white;
    cursor: pointer;
}

.input-area button:hover {
    background-color: #0056b3;
}
</style>