<script setup>
import { computed, ref } from 'vue'
import { toPng } from 'html-to-image'

const platform = ref('ios')
const sender = ref({ name: '小林', avatar: '🧑🏻‍💻', image: '' })
const receiver = ref({ name: '阿禾', avatar: '🌿', image: '' })
const activeRole = ref('sender')
const draft = ref('')
const emojiOpen = ref(false)
const voiceMode = ref(false)
const avatarInput = ref(null)
const phonePreview = ref(null)
const exporting = ref(false)
const messages = ref([
  { role: 'receiver', text: '嗨！这是一个公众号聊天界面 Demo 👋', time: '10:24', showTime: true },
  { role: 'sender', text: '看起来很不错，我们开始吧！', time: '10:25', showTime: false },
  { role: 'receiver', text: '头像和昵称都可以自由设置啦 ✨', time: '10:32', showTime: true },
])
const emojis = ['😀', '😂', '🥹', '😍', '🤔', '👍', '🎉', '✨', '🌿', '❤️']

const activeProfile = computed(() => activeRole.value === 'sender' ? sender.value : receiver.value)
function sendMessage() {
  const text = draft.value.trim()
  if (!text) return
  const now = new Date()
  const previous = messages.value.at(-1)
  const time = now.toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' })
  messages.value.push({ role: activeRole.value, text, time, showTime: !previous || previous.time !== time })
  draft.value = ''
  emojiOpen.value = false
}
function addEmoji(emoji) { draft.value += emoji }
function avatarFor(role) { return role === 'sender' ? sender.value.avatar : receiver.value.avatar }
function avatarImageFor(role) { return role === 'sender' ? sender.value.image : receiver.value.image }
function chooseAvatar() { avatarInput.value?.click() }
function uploadAvatar(event) {
  const file = event.target.files?.[0]
  if (!file) return
  if (activeProfile.value.image.startsWith('blob:')) URL.revokeObjectURL(activeProfile.value.image)
  activeProfile.value.image = URL.createObjectURL(file)
  event.target.value = ''
}
async function exportImage() {
  if (!phonePreview.value || exporting.value) return
  exporting.value = true
  try {
    const dataUrl = await toPng(phonePreview.value, {
      pixelRatio: 3,
      cacheBust: true,
      backgroundColor: '#ededeb',
    })
    const link = document.createElement('a')
    const safeName = receiver.value.name.trim().replace(/[\\/:*?"<>|]/g, '-') || '聊天'
    link.download = `${safeName}-微信聊天.png`
    link.href = dataUrl
    link.click()
  } catch (error) {
    window.alert('图片导出失败，请重试。')
    console.error(error)
  } finally {
    exporting.value = false
  }
}
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
        <div ref="phonePreview" class="phone" :class="platform">
          <div class="status-bar">
            <span class="status-time">9:41</span>
            <div class="status-icons" aria-label="手机状态">
              <svg viewBox="0 0 18 12"><path d="M1 11h2V8H1zm4 0h2V5H5zm4 0h2V2H9zm4 0h2V0h-2z"/></svg>
              <svg class="wifi" viewBox="0 0 16 12"><path d="M1 4.2A10.8 10.8 0 0 1 15 4.2M3.5 7a7 7 0 0 1 9 0M6.3 9.6a2.7 2.7 0 0 1 3.4 0"/><circle cx="8" cy="11" r="1"/></svg>
              <span v-if="platform === 'android'" class="network">5G</span>
              <span class="battery"><i></i></span>
            </div>
          </div>
          <div class="chat-nav"><button class="nav-icon back" aria-label="返回"><svg viewBox="0 0 24 24"><path d="m15 4-8 8 8 8"/></svg></button><strong>{{ receiver.name }}</strong><button class="nav-icon more" aria-label="更多"><i></i><i></i><i></i></button></div>
          <div class="chat-body">
            <template v-for="(message, index) in messages" :key="index">
              <div v-if="message.showTime" class="time-divider">{{ message.time }}</div>
              <div class="message-row" :class="message.role">
                <div class="avatar"><img v-if="avatarImageFor(message.role)" :src="avatarImageFor(message.role)" alt="" /><span v-else>{{ avatarFor(message.role) }}</span></div>
                <div class="message-content"><div class="bubble">{{ message.text }}</div></div>
              </div>
            </template>
          </div>
          <div class="composer">
            <button class="tool-btn" :class="{ active: voiceMode }" aria-label="切换语音输入" @click="voiceMode = !voiceMode"><svg viewBox="0 0 24 24"><path d="M8.5 8.5a5 5 0 0 1 0 7M5.5 5.5a9 9 0 0 1 0 13M12 11v2M15.5 8.5a5 5 0 0 0 0 7"/></svg></button>
            <button v-if="voiceMode" class="voice-input">按住 说话</button>
            <input v-else v-model="draft" @keyup.enter="sendMessage" placeholder="输入消息..." />
            <button class="tool-btn" aria-label="选择表情" @click="emojiOpen = !emojiOpen"><svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="9"/><circle class="fill" cx="9" cy="10" r="1"/><circle class="fill" cx="15" cy="10" r="1"/><path d="M8 14.5c1.1 2 6.9 2 8 0"/></svg></button>
            <button v-if="draft" class="send-btn" @click="sendMessage">发送</button>
            <button v-else class="tool-btn" aria-label="更多功能"><svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="9"/><path d="M12 7v10M7 12h10"/></svg></button>
          </div>
          <div v-if="emojiOpen" class="emoji-panel"><button v-for="emoji in emojis" :key="emoji" @click="addEmoji(emoji)">{{ emoji }}</button></div>
          <div class="home-indicator"></div>
        </div>
      </div>

      <aside class="controls">
        <div class="control-heading"><span>角色设置</span><span class="live-pill">LIVE PREVIEW</span></div>
        <div class="role-tabs"><button :class="{ selected: activeRole === 'sender' }" @click="activeRole = 'sender'">发送方</button><button :class="{ selected: activeRole === 'receiver' }" @click="activeRole = 'receiver'">接收方</button></div>
        <div class="avatar-setting">
          <span class="field-label">头像</span>
          <div class="avatar-options">
            <button class="avatar-preview" @click="chooseAvatar"><img v-if="activeProfile.image" :src="activeProfile.image" alt="当前头像" /><span v-else>{{ activeProfile.avatar }}</span><i>更换</i></button>
            <div><button class="upload-btn" @click="chooseAvatar">上传图片</button><small>支持 JPG、PNG、WebP</small></div>
          </div>
          <input ref="avatarInput" class="file-input" type="file" accept="image/png,image/jpeg,image/webp" @change="uploadAvatar" />
        </div>
        <label>或使用 Emoji<input v-model="activeProfile.avatar" maxlength="2" /></label>
        <label>显示昵称<input v-model="activeProfile.name" maxlength="10" /></label>
        <div class="hint">点击下方发送方 / 接收方切换编辑对象，修改会即时反映在预览中。</div>
        <button class="export-btn" :disabled="exporting" @click="exportImage">
          <svg viewBox="0 0 24 24"><path d="M12 3v12m0 0 4-4m-4 4-4-4M5 16v3a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2v-3"/></svg>
          {{ exporting ? '正在生成图片…' : '导出聊天图片' }}
        </button>
        <div class="feature-list"><div><span>✦</span><p><b>真实聊天体验</b><small>支持文字、Emoji 与时间戳</small></p></div><div><span>◉</span><p><b>双端 UI 适配</b><small>一键切换 iOS / Android 风格</small></p></div><div><span>⌁</span><p><b>轻量易部署</b><small>Vite 构建，可直接部署到 Cloudflare</small></p></div></div>
      </aside>
    </section>
    <footer>Built with Vue + Vite <span>·</span> Ready for Cloudflare Pages</footer>
  </main>
</template>
