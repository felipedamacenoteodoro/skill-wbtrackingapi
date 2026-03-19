# Workflow: Build Chat Frontend

<required_reading>
**Read these reference files NOW:**
1. references/api-endpoints.md
2. references/helper-library.md
</required_reading>

<process>

## Step 1: Setup environment variables

For Vite (React):
```env
VITE_WBTRACKINGAPI_URL=https://wbtrackingapi.com.br
VITE_WBTRACKINGAPI_TOKEN=your_token
```

For Next.js:
```env
NEXT_PUBLIC_WBTRACKINGAPI_URL=https://wbtrackingapi.com.br
NEXT_PUBLIC_WBTRACKINGAPI_TOKEN=your_token
```

## Step 2: Create the API helper (frontend version)

```typescript
// lib/whatsapp.ts
const API = import.meta.env.VITE_WBTRACKINGAPI_URL; // or process.env.NEXT_PUBLIC_WBTRACKINGAPI_URL
const TOKEN = import.meta.env.VITE_WBTRACKINGAPI_TOKEN;
const headers = { 'token': TOKEN, 'Content-Type': 'application/json' };

async function api(method: string, path: string, body?: any) {
  const res = await fetch(`${API}${path}`, {
    method, headers,
    body: body ? JSON.stringify(body) : undefined,
  });
  return res.json();
}

export const sendText = (phone: string, text: string) =>
  api('POST', '/chat/send/text', { Phone: phone, Body: text });

export const sendImage = (phone: string, base64: string, caption = '') =>
  api('POST', '/chat/send/image', { Phone: phone, Image: base64, Caption: caption });

export const sendDocument = (phone: string, base64: string, fileName: string, caption = '') =>
  api('POST', '/chat/send/document', { Phone: phone, Document: base64, FileName: fileName, Caption: caption });

export const sendAudio = (phone: string, base64: string) =>
  api('POST', '/chat/send/audio', { Phone: phone, Audio: base64, Ptt: true });

export const setTyping = (phone: string) =>
  api('POST', '/chat/presence', { Phone: phone, State: 'composing' });

export const markRead = (phone: string, ids: string[]) =>
  api('POST', '/chat/markread', { Id: ids, Chat: `${phone}@s.whatsapp.net` });

export const checkStatus = () => api('GET', '/session/status');
export const getContacts = () => api('GET', '/user/contacts');
export const getAvatar = (phone: string) => api('POST', '/user/avatar', { Phone: phone, Preview: false });

export function fileToBase64(file: File): Promise<string> {
  return new Promise((resolve) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result as string);
    reader.readAsDataURL(file);
  });
}
```

## Step 3: Build the Chat component

```tsx
// components/Chat.tsx
import { useState, useRef, useEffect, useCallback } from 'react';
import { sendText, sendImage, sendDocument, setTyping, fileToBase64 } from '../lib/whatsapp';

interface Message {
  id: string;
  text: string;
  fromMe: boolean;
  type: 'text' | 'image' | 'document' | 'audio';
  timestamp: Date;
  fileName?: string;
}

interface ChatProps {
  phone: string;
  contactName?: string;
}

export function Chat({ phone, contactName }: ChatProps) {
  const [messages, setMessages] = useState<Message[]>([]);
  const [input, setInput] = useState('');
  const [sending, setSending] = useState(false);
  const messagesEnd = useRef<HTMLDivElement>(null);
  const fileInput = useRef<HTMLInputElement>(null);

  useEffect(() => {
    messagesEnd.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  const addMessage = useCallback((msg: Partial<Message>) => {
    setMessages(prev => [...prev, {
      id: crypto.randomUUID(),
      text: '',
      fromMe: true,
      type: 'text',
      timestamp: new Date(),
      ...msg,
    }]);
  }, []);

  const handleSend = async () => {
    const text = input.trim();
    if (!text || sending) return;

    setSending(true);
    setInput('');
    addMessage({ text, fromMe: true, type: 'text' });

    try {
      await setTyping(phone);
      await sendText(phone, text);
    } catch (err) {
      console.error('Failed to send:', err);
    }
    setSending(false);
  };

  const handleFile = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    setSending(true);
    const base64 = await fileToBase64(file);
    const isImage = file.type.startsWith('image/');

    addMessage({
      text: isImage ? '📷 Image' : `📎 ${file.name}`,
      fromMe: true,
      type: isImage ? 'image' : 'document',
      fileName: file.name,
    });

    try {
      if (isImage) {
        await sendImage(phone, base64, file.name);
      } else {
        await sendDocument(phone, base64, file.name);
      }
    } catch (err) {
      console.error('Failed to send file:', err);
    }

    setSending(false);
    if (fileInput.current) fileInput.current.value = '';
  };

  return (
    <div className="chat-container">
      {/* Header */}
      <div className="chat-header">
        <div className="chat-header-avatar">
          {(contactName || phone).charAt(0).toUpperCase()}
        </div>
        <div>
          <div className="chat-header-name">{contactName || phone}</div>
          <div className="chat-header-status">Online</div>
        </div>
      </div>

      {/* Messages */}
      <div className="chat-messages">
        {messages.map((msg) => (
          <div key={msg.id} className={`chat-bubble ${msg.fromMe ? 'sent' : 'received'}`}>
            <span>{msg.text}</span>
            <span className="chat-time">
              {msg.timestamp.toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit' })}
            </span>
          </div>
        ))}
        <div ref={messagesEnd} />
      </div>

      {/* Input */}
      <div className="chat-input-bar">
        <button className="chat-attach-btn" onClick={() => fileInput.current?.click()}>
          📎
        </button>
        <input type="file" ref={fileInput} onChange={handleFile} hidden />
        <input
          className="chat-input"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyDown={(e) => e.key === 'Enter' && !e.shiftKey && handleSend()}
          placeholder="Digite uma mensagem..."
          disabled={sending}
        />
        <button className="chat-send-btn" onClick={handleSend} disabled={sending || !input.trim()}>
          ➤
        </button>
      </div>
    </div>
  );
}
```

## Step 4: Add styles

```css
/* styles/chat.css */
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  max-width: 600px;
  margin: 0 auto;
  background: #09090b;
  color: #fafafa;
}

.chat-header {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: rgba(255,255,255,0.03);
  border-bottom: 1px solid rgba(255,255,255,0.06);
}

.chat-header-avatar {
  width: 40px; height: 40px;
  border-radius: 50%;
  background: linear-gradient(135deg, #8b5cf6, #06b6d4);
  display: flex; align-items: center; justify-content: center;
  font-weight: 700; font-size: 1.1rem;
}

.chat-header-name { font-weight: 600; font-size: 0.95rem; }
.chat-header-status { font-size: 0.75rem; color: rgba(255,255,255,0.4); }

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.chat-bubble {
  max-width: 75%;
  padding: 8px 14px;
  border-radius: 16px;
  font-size: 0.9rem;
  line-height: 1.4;
  display: flex;
  flex-direction: column;
}

.chat-bubble.sent {
  align-self: flex-end;
  background: #8b5cf6;
  border-bottom-right-radius: 4px;
}

.chat-bubble.received {
  align-self: flex-start;
  background: rgba(255,255,255,0.08);
  border-bottom-left-radius: 4px;
}

.chat-time {
  font-size: 0.65rem;
  color: rgba(255,255,255,0.4);
  align-self: flex-end;
  margin-top: 2px;
}

.chat-input-bar {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  background: rgba(255,255,255,0.03);
  border-top: 1px solid rgba(255,255,255,0.06);
}

.chat-input {
  flex: 1;
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 20px;
  padding: 10px 16px;
  color: #fafafa;
  font-size: 0.9rem;
  outline: none;
}

.chat-input:focus { border-color: #8b5cf6; }
.chat-input::placeholder { color: rgba(255,255,255,0.25); }

.chat-attach-btn, .chat-send-btn {
  background: none;
  border: none;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 8px;
  border-radius: 50%;
  transition: background 0.15s;
}

.chat-attach-btn:hover, .chat-send-btn:hover {
  background: rgba(255,255,255,0.06);
}

.chat-send-btn:disabled { opacity: 0.3; cursor: default; }
```

## Step 5: Receive messages via webhook (server-side)

The frontend needs a backend to receive messages. Use SSE or WebSocket to push received messages to the frontend:

```javascript
// server.js — add SSE endpoint for real-time messages
const clients = new Map();

app.get('/events/:phone', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  clients.set(req.params.phone, res);
  req.on('close', () => clients.delete(req.params.phone));
});

// In webhook handler, push to SSE:
app.post('/webhook', (req, res) => {
  res.sendStatus(200);
  const phone = getPhone(req.body);
  const client = clients.get(phone);
  if (client) {
    client.write(`data: ${JSON.stringify(req.body)}\n\n`);
  }
});
```

```tsx
// In React component, connect to SSE:
useEffect(() => {
  const events = new EventSource(`/events/${phone}`);
  events.onmessage = (e) => {
    const webhook = JSON.parse(e.data);
    if (webhook.type === 'Message' && !isFromMe(webhook)) {
      addMessage({ text: getText(webhook), fromMe: false, type: getMessageType(webhook) });
    }
  };
  return () => events.close();
}, [phone]);
```

</process>

<success_criteria>
This workflow is complete when:
- [ ] Chat component renders with header, messages, and input bar
- [ ] Text messages can be sent via the input
- [ ] Files (images, documents) can be sent via the attach button
- [ ] Environment variables are configured
- [ ] Styles match the dark theme
- [ ] (Optional) Real-time message receiving via SSE works
</success_criteria>

<anti_patterns>
- **Exposing token in client-side code**: For production, proxy API calls through your backend
- **Missing file type detection**: Always check file.type to determine send endpoint
- **Not clearing file input**: Reset fileInput.current.value after sending
- **Blocking UI during send**: Use loading states, don't freeze the input
</anti_patterns>
