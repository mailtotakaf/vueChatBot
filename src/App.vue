<script setup>
import { ref } from 'vue'

// メッセージのリスト（初期データ）
const messages = ref([
  { id: 1, text: 'こんにちは！何かお手伝いしましょうか？', isBot: true },
])

// 入力中のテキスト
const inputText = ref('')

// 送信ボタンを押した時の処理
const sendMessage = () => {
  if (!inputText.value.trim()) return

  const userText = inputText.value

  // 1. ユーザーのメッセージを追加
  messages.value.push({
    id: Date.now(),
    // text: inputText.value,
    text: userText,
    isBot: false
  })

  // 2. AIの返答（仮）
  setTimeout(() => {
    messages.value.push({
      id: Date.now() + 1,
      text: '「' + userText + '」って言いましたね！',
      isBot: true
    })
  }, 1000)

  // 入力欄を空にする
  inputText.value = ''
}
</script>

<template>
  <div class="chat-container">
    <header>AI Chat Bot</header>

    <div class="chat-log">
      <div v-for="msg in messages" :key="msg.id" :class="['message', msg.isBot ? 'bot' : 'user']">
        <div class="bubble">{{ msg.text }}</div>
      </div>
    </div>

    <div class="input-area">
      <input 
        v-model="inputText" 
        @keyup.enter="sendMessage" 
        placeholder="メッセージを入力..."
      />
      <button @click="sendMessage">送信</button>
    </div>
  </div>
</template>

<style scoped>
.chat-container {
  max-width: 500px;
  margin: 0 auto;
  height: 90vh;
  display: flex;
  flex-direction: column;
  border: 1px solid #ddd;
  background: #f9f9f9;
}

header {
  padding: 15px;
  background: #42b983; /* Vueカラー */
  color: white;
  text-align: center;
  font-weight: bold;
}

.chat-log {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.message {
  display: flex;
}

.message.user { justify-content: flex-end; }
.message.bot { justify-content: flex-start; }

.bubble {
  max-width: 70%;
  padding: 10px 15px;
  border-radius: 15px;
  font-size: 14px;
  line-height: 1.4;
}

.user .bubble {
  background: #42b983;
  color: white;
  border-bottom-right-radius: 2px;
}

.bot .bubble {
  background: #eee;
  color: #333;
  border-bottom-left-radius: 2px;
}

.input-area {
  padding: 15px;
  display: flex;
  gap: 10px;
  background: white;
  border-top: 1px solid #ddd;
}

input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  outline: none;
}

button {
  padding: 0 20px;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>