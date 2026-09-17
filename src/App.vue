<script setup>
import { computed, ref } from 'vue'

const platform = ref('ios')
const sender = ref({ name: '小林', avatar: '🧑🏻‍💻' })
const receiver = ref({ name: '阿禾', avatar: '🌿' })
const activeRole = ref('sender')
const draft = ref('')
const emojiOpen = ref(false)
const messages = ref([
  { role: 'receiver', text: '嗨！这是一个公众号聊天界面 Demo 👋', time: '10:24' },
  { role: 'sender', text: '看起来很不错，我们开始吧！', time: '10:25' },
])
const emojis = ['😀', '😂', '🥹', '😍', '🤔', '👍', '🎉', '✨', '🌿', '❤️']

const activeProfile = computed(() => activeRole.value === 'sender' ? sender.value : receiver.value)
function sendMessage() {
  const text = draft.value.trim()
  if (!text) return
  messages.value.push({ role: activeRole.value, text, time: new Date().toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' }) })
  draft.value = ''
  emojiOpen.value = false
}
function addEmoji(emoji) { draft.value += emoji }
function avatarFor(role) { return role === 'sender' ? sender.value.avatar : receiver.value.avatar }
function nameFor(role) { return role === 'sender' ? sender.value.name : receiver.value.name }
</script>

<template>
  <main class="page-shell">
    <section class="intro">
      <div class="eyebrow"><span class="dot"></span> WeChat Chat Simulator</div>
      <h1>公众号聊天<br /><em>轻松做个 Demo。</em></h1>
      <p>一个可以自由编辑角色与消息的聊天预览工具，适合产品原型、设计展示与内容演示。</p>
      <div class="platform-switch">
        <button :class="{ active: platform === 'ios' }" @click="platform = 'ios'"> iOS</button>
        <button :class="{ active: platform === 'android' }" @click="platform = 'android'">Android</button>
      </div>
    </section>

    <section class="workspace">
      <div class="phone-wrap">
        <div class="phone" :class="platform">
          <div class="status-bar"><span>9:41</span><span class="status-icons">● ◔ ▮</span></div>
          <div class="chat-nav"><span class="back">‹</span><strong>{{ receiver.name }}</strong><span class="more">···</span></div>
          <div class="chat-body">
            <div v-for="(message, index) in messages" :key="index" class="message-row" :class="message.role">
              <div class="avatar">{{ avatarFor(message.role) }}</div>
              <div class="message-content"><span class="message-name">{{ nameFor(message.role) }}</span><div class="bubble">{{ message.text }}</div><small>{{ message.time }}</small></div>
            </div>
          </div>
          <div class="composer"><button class="round-btn">＋</button><input v-model="draft" @keyup.enter="sendMessage" placeholder="输入消息..." /><button class="emoji-btn" @click="emojiOpen = !emojiOpen">☺</button><button class="send-btn" @click="sendMessage">发送</button></div>
          <div v-if="emojiOpen" class="emoji-panel"><button v-for="emoji in emojis" :key="emoji" @click="addEmoji(emoji)">{{ emoji }}</button></div>
          <div class="home-indicator"></div>
        </div>
      </div>

      <aside class="controls">
        <div class="control-heading"><span>角色设置</span><span class="live-pill">LIVE PREVIEW</span></div>
        <div class="role-tabs"><button :class="{ selected: activeRole === 'sender' }" @click="activeRole = 'sender'">发送方</button><button :class="{ selected: activeRole === 'receiver' }" @click="activeRole = 'receiver'">接收方</button></div>
        <label>头像 Emoji<input v-model="activeProfile.avatar" maxlength="2" /></label>
        <label>显示昵称<input v-model="activeProfile.name" maxlength="10" /></label>
        <div class="hint">点击下方发送方 / 接收方切换编辑对象，修改会即时反映在预览中。</div>
        <div class="feature-list"><div><span>✦</span><p><b>真实聊天体验</b><small>支持文字、Emoji 与时间戳</small></p></div><div><span>◉</span><p><b>双端 UI 适配</b><small>一键切换 iOS / Android 风格</small></p></div><div><span>⌁</span><p><b>轻量易部署</b><small>Vite 构建，可直接部署到 Cloudflare</small></p></div></div>
      </aside>
    </section>
    <footer>Built with Vue + Vite <span>·</span> Ready for Cloudflare Pages</footer>
  </main>
</template>
