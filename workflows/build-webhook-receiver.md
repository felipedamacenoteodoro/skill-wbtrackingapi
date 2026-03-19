# Workflow: Build Webhook Receiver

<required_reading>
**Read these reference files NOW:**
1. references/webhook-payload.md
2. references/helper-library.md
</required_reading>

<process>

## Step 1: Create the helper library

Copy the helper module from `references/helper-library.md` into `lib/wbtrackingapi.js`.

## Step 2: Create the webhook server

**Node.js with Express:**

```javascript
import express from 'express';
import { getPhone, getText, getMessageType, isFromMe, isGroup, getPushName, getMessageId, getTracking, isFromAd, sendText, setTyping, markRead } from './lib/wbtrackingapi.js';

const app = express();
app.use(express.json({ limit: '50mb' }));

app.post('/webhook', async (req, res) => {
  // CRITICAL: respond immediately
  res.sendStatus(200);

  const webhook = req.body;

  // Only process incoming messages
  if (webhook.type !== 'Message') return;
  if (isFromMe(webhook)) return;

  const phone = getPhone(webhook);
  const text = getText(webhook);
  const type = getMessageType(webhook);
  const name = getPushName(webhook);
  const msgId = getMessageId(webhook);
  const tracking = getTracking(webhook);

  console.log(`[${type}] ${name} (${phone}): ${text}`);

  // Log tracking if from ad
  if (isFromAd(webhook)) {
    console.log(`[UTM] Campaign: ${tracking.utm_campaign} | Ad: ${tracking.utm_content}`);
  }

  // Mark as read
  await markRead(phone, msgId);

  // Show typing
  await setTyping(phone);

  // Process based on message type
  switch (type) {
    case 'text':
      await handleTextMessage(phone, text, tracking);
      break;
    case 'image':
      await handleMediaMessage(phone, type, msgId);
      break;
    case 'document':
      await handleMediaMessage(phone, type, msgId);
      break;
    case 'audio':
      await handleMediaMessage(phone, type, msgId);
      break;
    default:
      await sendText(phone, `Recebi sua mensagem do tipo: ${type}`);
  }
});

async function handleTextMessage(phone, text, tracking) {
  // YOUR LOGIC HERE
  await sendText(phone, `Recebi: "${text}"`);
}

async function handleMediaMessage(phone, type, msgId) {
  // YOUR LOGIC HERE
  await sendText(phone, `Recebi seu ${type}. Vou processar!`);
}

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Webhook server running on port ${PORT}`));
```

**Python with FastAPI:**

```python
from fastapi import FastAPI, Request, BackgroundTasks
import httpx
import os

app = FastAPI()
API = os.getenv("WBTRACKINGAPI_URL")
TOKEN = os.getenv("WBTRACKINGAPI_TOKEN")
HEADERS = {"token": TOKEN, "Content-Type": "application/json"}

def get_phone(wh): return wh["event"]["Info"]["MessageSource"]["Sender"].replace("@s.whatsapp.net", "")
def get_text(wh):
    m = wh["event"]["Message"]
    return m.get("conversation") or (m.get("extendedTextMessage") or {}).get("text") or (m.get("imageMessage") or {}).get("caption") or ""
def is_from_me(wh): return wh["event"]["Info"]["MessageSource"].get("IsFromMe", False)

async def process(phone, text, tracking):
    async with httpx.AsyncClient() as c:
        await c.post(f"{API}/chat/presence", headers=HEADERS, json={"Phone": phone, "State": "composing"})
        await c.post(f"{API}/chat/send/text", headers=HEADERS, json={"Phone": phone, "Body": f"Recebi: {text}"})

@app.post("/webhook")
async def webhook(request: Request, bg: BackgroundTasks):
    data = await request.json()
    if data.get("type") != "Message" or is_from_me(data): return {"ok": True}
    phone = get_phone(data)
    text = get_text(data)
    tracking = data.get("tracking", {})
    bg.add_task(process, phone, text, tracking)
    return {"ok": True}
```

## Step 3: Add environment variables

Create `.env`:
```
WBTRACKINGAPI_URL=https://wbtrackingapi.com.br
WBTRACKINGAPI_TOKEN=your_token_here
PORT=3000
```

## Step 4: Expose with ngrok (development)

```bash
ngrok http 3000
```

Then configure the webhook in WBTrackinAPI:
```javascript
import { setWebhook } from './lib/wbtrackingapi.js';
// Use apenas "Message" — é o único evento necessário para receber mensagens
// Os demais eventos (ReadReceipt, Presence, etc.) são opcionais
await setWebhook('https://your-ngrok-url.ngrok.io/webhook', ['Message']);
```

</process>

<success_criteria>
This workflow is complete when:
- [ ] Webhook server starts without errors
- [ ] Server responds with 200 immediately on POST /webhook
- [ ] Messages are processed only when type=Message and IsFromMe=false
- [ ] Tracking data is extracted from the webhook payload
- [ ] The server can send replies back via the API
</success_criteria>

<anti_patterns>
- **Blocking the webhook response**: Never do heavy processing before sending 200
- **Missing IsFromMe check**: Will cause infinite message loops
- **Hardcoding credentials**: Always use environment variables
- **Using Authorization header**: The header is `token` (lowercase), not `Authorization`
- **Adding + to phone numbers**: Numbers must not have + prefix
</anti_patterns>
