<overview>
Ready-to-use JavaScript/TypeScript helper module for WBTrackinAPI.
Copy this file as `lib/wbtrackingapi.js` or `lib/wbtrackingapi.ts` in the user's project.
</overview>

<helper_module>

```javascript
// lib/wbtrackingapi.js
// WBTrackinAPI Helper Library

const API_URL = process.env.WBTRACKINGAPI_URL;
const TOKEN = process.env.WBTRACKINGAPI_TOKEN;

if (!API_URL) throw new Error('WBTRACKINGAPI_URL environment variable is required');
if (!TOKEN) throw new Error('WBTRACKINGAPI_TOKEN environment variable is required');

const headers = { 'token': TOKEN, 'Content-Type': 'application/json' };

async function api(method, path, body = null) {
  const opts = { method, headers };
  if (body) opts.body = JSON.stringify(body);
  const res = await fetch(`${API_URL}${path}`, opts);
  return res.json();
}

// ===== SENDING MESSAGES =====

export const sendText = (phone, text) =>
  api('POST', '/chat/send/text', { Phone: phone, Body: text });

export const sendImage = (phone, base64, caption = '') =>
  api('POST', '/chat/send/image', { Phone: phone, Image: base64, Caption: caption });

export const sendDocument = (phone, base64, fileName, caption = '') =>
  api('POST', '/chat/send/document', { Phone: phone, Document: base64, FileName: fileName, Caption: caption });

export const sendAudio = (phone, base64, ptt = true) =>
  api('POST', '/chat/send/audio', { Phone: phone, Audio: base64, Ptt: ptt });

export const sendVideo = (phone, base64, caption = '') =>
  api('POST', '/chat/send/video', { Phone: phone, Video: base64, Caption: caption });

export const sendLocation = (phone, lat, lng, name = '') =>
  api('POST', '/chat/send/location', { Phone: phone, Latitude: lat, Longitude: lng, Name: name });

export const sendContact = (phone, contactName, contactPhone) =>
  api('POST', '/chat/send/contact', { Phone: phone, ContactName: contactName, ContactPhone: contactPhone });

export const sendButtons = (phone, title, buttons) =>
  api('POST', '/chat/send/buttons', { Phone: phone, Title: title, Buttons: buttons.map(b => ({ ButtonId: b.id, ButtonText: b.text })) });

export const sendList = (phone, title, description, buttonText, sections) =>
  api('POST', '/chat/send/list', { Phone: phone, Title: title, Description: description, ButtonText: buttonText, Sections: sections });

// ===== CHAT ACTIONS =====

export const setTyping = (phone) =>
  api('POST', '/chat/presence', { Phone: phone, State: 'composing' });

export const setPaused = (phone) =>
  api('POST', '/chat/presence', { Phone: phone, State: 'paused' });

export const setRecording = (phone) =>
  api('POST', '/chat/presence', { Phone: phone, State: 'recording' });

export const markRead = (phone, messageIds) =>
  api('POST', '/chat/markread', { Id: Array.isArray(messageIds) ? messageIds : [messageIds], Chat: `${phone}@s.whatsapp.net` });

export const react = (phone, messageId, emoji) =>
  api('POST', '/chat/react', { Phone: phone, Id: messageId, Body: emoji });

export const deleteMessage = (phone, messageId) =>
  api('POST', '/chat/delete', { Phone: phone, Id: messageId });

// ===== MEDIA DOWNLOAD =====

export const downloadImage = (phone, messageId) =>
  api('POST', '/chat/downloadimage', { Phone: phone, Id: messageId });

export const downloadVideo = (phone, messageId) =>
  api('POST', '/chat/downloadvideo', { Phone: phone, Id: messageId });

export const downloadAudio = (phone, messageId) =>
  api('POST', '/chat/downloadaudio', { Phone: phone, Id: messageId });

export const downloadDocument = (phone, messageId) =>
  api('POST', '/chat/downloaddocument', { Phone: phone, Id: messageId });

// ===== SESSION =====

export const checkStatus = () => api('GET', '/session/status');
export const connect = () => api('POST', '/session/connect', { Subscribe: ['All'], Immediate: true });
export const getQR = () => api('GET', '/session/qr');
export const pairPhone = (phone) => api('POST', '/session/pairphone', { Phone: phone });
export const logout = () => api('POST', '/session/logout');

// ===== CONTACTS =====

export const checkWhatsApp = (phones) =>
  api('POST', '/user/check', { Phone: Array.isArray(phones) ? phones : [phones] });

export const getContacts = () => api('GET', '/user/contacts');

export const getAvatar = (phone) =>
  api('POST', '/user/avatar', { Phone: phone, Preview: false });

export const getUserInfo = (phone) =>
  api('POST', '/user/info', { Phone: [`${phone}@s.whatsapp.net`] });

// ===== META ADS =====

export const setMetaAdsToken = (metaToken) =>
  api('POST', '/session/meta-ads/config', { meta_ads_token: metaToken });

export const getMetaAdsConfig = () => api('GET', '/session/meta-ads/config');

export const deleteMetaAdsConfig = () => api('DELETE', '/session/meta-ads/config');

export const testMetaAdsToken = (metaToken) =>
  api('POST', '/session/meta-ads/test', { token: metaToken });

// ===== WEBHOOK =====

// Use ['Message'] para receber mensagens. Outros eventos são opcionais.
export const setWebhook = (url, events = ['Message']) =>
  api('POST', '/webhook', { WebhookURL: url, Events: events });

export const getWebhook = () => api('GET', '/webhook');
export const deleteWebhook = () => api('DELETE', '/webhook');

// ===== GROUPS =====

export const listGroups = () => api('GET', '/group/list');
export const createGroup = (name, participants) =>
  api('POST', '/group/create', { Name: name, Participants: participants.map(p => p.includes('@') ? p : `${p}@s.whatsapp.net`) });

// ===== WEBHOOK DATA EXTRACTORS =====

export const getPhone = (wh) => (wh.event?.Info?.MessageSource?.Sender || '').replace('@s.whatsapp.net', '');
export const getText = (wh) => { const m = wh.event?.Message; return m?.conversation || m?.extendedTextMessage?.text || m?.imageMessage?.caption || m?.videoMessage?.caption || m?.documentMessage?.caption || ''; };
export const getMessageType = (wh) => { const m = wh.event?.Message; if (!m) return 'unknown'; if (m.conversation || m.extendedTextMessage) return 'text'; if (m.imageMessage) return 'image'; if (m.audioMessage) return 'audio'; if (m.videoMessage) return 'video'; if (m.documentMessage) return 'document'; if (m.stickerMessage) return 'sticker'; if (m.locationMessage) return 'location'; if (m.contactMessage) return 'contact'; return 'unknown'; };
export const isFromMe = (wh) => wh.event?.Info?.MessageSource?.IsFromMe || false;
export const isGroup = (wh) => wh.event?.Info?.MessageSource?.IsGroup || false;
export const getPushName = (wh) => wh.event?.Info?.PushName || '';
export const getMessageId = (wh) => wh.event?.Info?.ID || '';
export const getTracking = (wh) => wh.tracking || { utm_source: 'organico', utm_campaign: '-', utm_medium: '-', utm_content: '-', utm_platform: '-' };
export const isFromAd = (wh) => wh.tracking?.utm_source === 'FaceAds';
```

</helper_module>

<usage_instructions>

When building a project for the user, ALWAYS:

1. Create `lib/wbtrackingapi.js` with the module above
2. Add env vars to `.env.example`:
   ```
   WBTRACKINGAPI_URL=https://wbtrackingapi.com.br
   WBTRACKINGAPI_TOKEN=your_instance_token
   ```
3. Import functions from the helper: `import { sendText, getPhone, getText } from './lib/wbtrackingapi.js'`
4. For React frontends, prefix env vars with `VITE_` or `NEXT_PUBLIC_` accordingly

</usage_instructions>
