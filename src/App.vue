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
  background: #f9f9f9;
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
  display: block;      /* flexの干渉を防ぐ */
  width: 100%;         /* 横幅はいっぱい */
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  outline: none;

  /* --- ここが重要 --- */
  resize: vertical !important; /* 強制的に上下リサイズを許可 */
  overflow: auto;              /* スクロールバーも出るように */
  min-height: 44px;            /* 最小の高さ */
  height: 60px;               /* 初期の高さ（これがないとつまみが出にくい） */
  max-height: none;            /* 限界をなくす */
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
</style>

<template>
  <div class="chat-container">
    <header>AI Chat Bot</header>

    <div class="chat-log" ref="chatLogRef">
      <div v-for="msg in messages" :key="msg.id" :class="['message', msg.isBot ? 'bot' : 'user']">
        <div class="bubble">{{ msg.text }}</div>
      </div>
    </div>

    <div class="input-area">
      <textarea v-model="inputText" placeholder="メッセージを入力..." rows="1" ref="textareaRef"></textarea>
      <button @click="sendMessage">送信</button>
    </div>
  </div>
</template>

<style scoped>
.chat-container {
  /* 固定の height ではなく、ビューポートの高さ(dvh)を使うとスマホで安定します */
  height: 100dvh;
  max-width: 500px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  background: #f9f9f9;
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
  /* ↓ これを追加：APIから返ってきた改行コード(\n)を有効にする */
  white-space: pre-wrap; 
  word-wrap: break-word;
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
  flex: 1;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  outline: none;
  resize: none;
  /* ユーザーが手動でサイズ変更できないようにする */
  font-family: inherit;
  font-size: 16px;
  line-height: 1.5;
  max-height: 150px;
  /* 伸びすぎ防止 */
  min-height: 44px;
  /* 指でタップしやすい高さ */
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