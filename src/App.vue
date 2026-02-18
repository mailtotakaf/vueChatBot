<script setup>
import { ref, watch, nextTick } from 'vue'
import axios from 'axios' // axiosをインポート

// メッセージのリスト（初期データ）
const messages = ref([{ id: 1, text: '本物のAIに繋がりました！何でも聞いてください。', isBot: true }])

// 入力中のテキスト
const inputText = ref('')

// --- Difyの設定 ---
const DIFY_API_KEY = import.meta.env.VITE_DIFY_API_KEY
const DIFY_API_URL = 'https://api.dify.ai/v1/chat-messages'

// 送信ボタンを押した時の処理
const sendMessage = async () => {
  if (!inputText.value.trim()) return

  const userText = inputText.value

  // 1. ユーザーのメッセージを追加
  messages.value.push({
    id: Date.now(),
    // text: inputText.value,
    text: userText,
    isBot: false
  })
  inputText.value = ''

  // 2. AIの返答
  try {
    // Dify APIにリクエストを送る
    const response = await axios.post(DIFY_API_URL, {
      inputs: {},
      query: userText,
      response_mode: "blocking",
      user: "abc-123", // ユーザー識別用（適当でOK）
    }, {
      headers: {
        'Authorization': `Bearer ${DIFY_API_KEY}`,
        'Content-Type': 'application/json'
      }
    })

    // AIの返答を画面に追加
    messages.value.push({
      id: Date.now() + 1,
      text: response.data.answer,
      isBot: true
    })

  } catch (error) {
    console.error('エラーが出ちゃいました:', error)
    messages.value.push({
      id: Date.now() + 1,
      text: 'ごめんなさい、接続エラーです...',
      isBot: true
    })
  }
}

const isOpen = ref(false) // 窓が開いているかどうかの状態

const textareaRef = ref(null)

watch(messages, async () => {
  // メッセージが追加されて、DOMが更新されるのを待つ
  await nextTick()

  if (chatLogRef.value) {
    chatLogRef.value.scrollTo({
      top: chatLogRef.value.scrollHeight,
      behavior: 'smooth'
    })
  }
}, { deep: true })

const handleKeyDown = (e) => {
  // PCの場合（ShiftなしのEnter）だけ送信
  // スマホ（タッチデバイス）かどうかを簡易判定
  const isMobile = window.matchMedia("(max-width: 768px)").matches;

  if (e.key === 'Enter' && !e.shiftKey && !isMobile) {
    e.preventDefault();
    sendMessage();
  }
}

// メッセージ表示エリアを捕まえるためのリファレンス
const chatLogRef = ref(null)

// スクロールを一番下にする関数
const scrollToBottom = () => {
  console.log('スクロール実行！', chatLogRef.value.scrollHeight)
  nextTick(() => {
    if (chatLogRef.value) {
      chatLogRef.value.scrollTo({
        top: chatLogRef.value.scrollHeight,
        behavior: 'smooth' // 「シュッ」と滑らかに動かす
      })
    }
  })
}

// メッセージの配列(messages)の中身が増えたら、自動で関数を実行する
watch(messages, () => {
  scrollToBottom()
}, { deep: true }) // 配列の中身の変化までしっかり監視する設定
</script>

<style scoped>
.chat-container {
  /* 固定の height ではなく、ビューポートの高さ(dvh)を使うとスマホで安定します */
  height: 100dvh;
  max-width: 500px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  background: transparent;
}

header {
  padding: 15px;
  background: #42b983;
  /* Vueカラー */
  color: white;
  text-align: center;
  font-weight: bold;
}

.chat-log {
  flex: 1;
  /* 親のcontainerの中で目一杯広がる */
  overflow-y: auto;
  /* はみ出た分をスクロール可能にする */
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  /* 念のため */
  scroll-behavior: smooth;
}

.message {
  display: flex;
}

.message.user {
  justify-content: flex-end;
}

.message.bot {
  justify-content: flex-start;
}

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

textarea {
  display: block;
  /* flexの干渉を防ぐ */
  width: 100%;
  /* 横幅はいっぱい */
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  outline: none;

  /* --- ここが重要 --- */
  resize: vertical !important;
  /* 強制的に上下リサイズを許可 */
  overflow: auto;
  /* スクロールバーも出るように */
  min-height: 44px;
  /* 最小の高さ */
  height: 60px;
  /* 初期の高さ（これがないとつまみが出にくい） */
  max-height: none;
  /* 限界をなくす */
  /* ------------------ */

  font-family: inherit;
  font-size: 16px;
  line-height: 1.5;
}

button {
  padding: 0 20px;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

/* ウィジェット全体のコンテナ */
.chat-widget {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 9999;
  font-family: sans-serif;
}

/* 丸いボタン */
.launcher-btn {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: #42b983;
  color: white;
  border: none;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  font-size: 24px;
}

/* チャットウィンドウ */
.chat-window {
  position: absolute;
  bottom: 80px;
  /* ボタンより少し上に表示 */
  right: 0;
  width: 350px;
  height: 500px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* スマホ対応：画面が小さい時はもっと横幅を広げる */
@media (max-width: 480px) {
  .chat-window {
    width: calc(100vw - 40px);
    height: 70vh;
  }
}

/* アニメーション（ふわっと出す） */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s, transform 0.3s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(20px);
}
</style>

<template>
  <div class="chat-widget">
    <transition name="fade">
      <div v-if="isOpen" class="chat-window">
        <header>
          AIサポート
          <button @click="isOpen = false" class="close-btn">×</button>
        </header>

        <div class="chat-log" ref="chatLogRef">
          <div v-for="msg in messages" :key="msg.id" :class="['message', msg.isBot ? 'bot' : 'user']">
            <div class="bubble">{{ msg.text }}</div>
          </div>
        </div>

        <div class="input-area">
          <textarea v-model="inputText" @keydown.enter.exact.prevent="sendMessage" placeholder="メッセージを入力..."
            rows="1"></textarea>
          <button @click="sendMessage">送信</button>
        </div>
      </div>
    </transition>

    <button class="launcher-btn" @click="isOpen = !isOpen">
      <span v-if="!isOpen">💬</span>
      <span v-else>↓</span>
    </button>
  </div>
</template>

<style>
body, #app {
  background: transparent !important;
}
</style>