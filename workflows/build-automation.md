# Workflow: Build Message Automation

<required_reading>
**Read these reference files NOW:**
1. references/api-endpoints.md
2. references/webhook-payload.md
3. references/helper-library.md
</required_reading>

<process>

## Step 1: Create the helper library

Copy the helper module from `references/helper-library.md` into `lib/wbtrackingapi.js`.

## Step 2: Build the automation bot

```javascript
import express from 'express';
import { getPhone, getText, getMessageType, isFromMe, getPushName, getTracking, isFromAd, sendText, sendImage, sendDocument, setTyping, markRead } from './lib/wbtrackingapi.js';

const app = express();
app.use(express.json({ limit: '50mb' }));

// Simple delay helper
const wait = (ms) => new Promise(r => setTimeout(r, ms));

// Message queue to avoid rate limits
const queue = [];
let processing = false;

async function processQueue() {
  if (processing) return;
  processing = true;
  while (queue.length > 0) {
    const job = queue.shift();
    try {
      await job();
    } catch (err) {
      console.error('Queue job failed:', err);
    }
    await wait(1000); // 1 second between messages
  }
  processing = false;
}

function enqueue(fn) {
  queue.push(fn);
  processQueue();
}

// Auto-reply rules
const rules = [
  {
    match: /pre[cç]o|valor|quanto custa/i,
    reply: async (phone, name) => {
      await setTyping(phone);
      await wait(1500);
      await sendText(phone, `Olá ${name}! 😊\n\nNossos planos:\n\n📦 *Start* — R$ 97/mês (1 instância)\n📦 *Pro* — R$ 197/mês (5 instâncias)\n📦 *Business* — R$ 397/mês (15 instâncias)\n\nTodos incluem UTM Tracking do Meta Ads!\n\nQual plano te interessa?`);
    }
  },
  {
    match: /oi|olá|ola|bom dia|boa tarde|boa noite|hey|hello/i,
    reply: async (phone, name, tracking) => {
      await setTyping(phone);
      await wait(1000);
      let msg = `Olá ${name}! 👋\nSeja bem-vindo! Como posso ajudar?`;
      if (isFromAd({ tracking })) {
        msg += `\n\n_Vi que você veio pela campanha "${tracking.utm_campaign}". Tem alguma dúvida?_`;
      }
      await sendText(phone, msg);
    }
  },
  {
    match: /.*/,  // Fallback — catches everything else
    reply: async (phone, name) => {
      await setTyping(phone);
      await wait(1000);
      await sendText(phone, `Obrigado pela mensagem, ${name}! Um atendente vai te responder em breve. ⏳`);
    }
  }
];

app.post('/webhook', async (req, res) => {
  res.sendStatus(200);

  const wh = req.body;
  if (wh.type !== 'Message' || isFromMe(wh)) return;

  const phone = getPhone(wh);
  const text = getText(wh);
  const name = getPushName(wh) || 'Cliente';
  const tracking = getTracking(wh);
  const msgId = wh.event?.Info?.ID;

  console.log(`[MSG] ${name} (${phone}): ${text}`);

  // Mark as read
  enqueue(() => markRead(phone, msgId));

  // Find matching rule
  for (const rule of rules) {
    if (rule.match.test(text)) {
      enqueue(() => rule.reply(phone, name, tracking));
      break;
    }
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Bot running on port ${PORT}`));
```

## Step 3: Add bulk messaging (use with caution)

```javascript
// scripts/bulk-send.js
import { sendText, checkWhatsApp } from './lib/wbtrackingapi.js';

const contacts = [
  { phone: '5511999999999', name: 'João' },
  { phone: '5511888888888', name: 'Maria' },
];

const wait = (ms) => new Promise(r => setTimeout(r, ms));

async function bulkSend(message) {
  for (const contact of contacts) {
    // Check if number has WhatsApp
    const check = await checkWhatsApp(contact.phone);
    const hasWA = check.data?.[0]?.IsInWhatsapp;

    if (!hasWA) {
      console.log(`[SKIP] ${contact.phone} — not on WhatsApp`);
      continue;
    }

    const personalizedMsg = message.replace('{name}', contact.name);
    const result = await sendText(contact.phone, personalizedMsg);

    if (result.success) {
      console.log(`[OK] ${contact.phone}`);
    } else {
      console.log(`[FAIL] ${contact.phone}: ${result.error}`);
    }

    // IMPORTANT: wait between messages to avoid ban
    await wait(3000 + Math.random() * 2000); // 3-5 seconds
  }
}

bulkSend('Olá {name}! Temos novidades pra você 🎉');
```

## Step 4: Scheduled messages

```javascript
// scripts/scheduled.js
import { sendText } from './lib/wbtrackingapi.js';

// Send a message at a specific time
function scheduleMessage(phone, text, date) {
  const delay = date.getTime() - Date.now();
  if (delay <= 0) {
    console.error('Date must be in the future');
    return;
  }
  console.log(`Scheduled message to ${phone} in ${Math.round(delay / 1000)}s`);
  setTimeout(async () => {
    const result = await sendText(phone, text);
    console.log(`Sent to ${phone}:`, result.success);
  }, delay);
}

// Example: send tomorrow at 9am
const tomorrow9am = new Date();
tomorrow9am.setDate(tomorrow9am.getDate() + 1);
tomorrow9am.setHours(9, 0, 0, 0);

scheduleMessage('5511999999999', 'Bom dia! Lembrete do seu agendamento hoje às 14h.', tomorrow9am);
```

</process>

<success_criteria>
This workflow is complete when:
- [ ] Bot receives messages via webhook and auto-replies based on rules
- [ ] Messages are queued with delays to avoid rate limits
- [ ] Tracking data from Meta Ads is captured
- [ ] IsFromMe check prevents loops
- [ ] Composing presence is set before replies
</success_criteria>

<anti_patterns>
- **No delay between bulk messages**: WhatsApp will ban you. Minimum 3 seconds between messages.
- **Sending identical messages**: Vary the content slightly to avoid spam detection.
- **Not checking IsInWhatsapp before sending**: Wastes API calls and may cause errors.
- **Replying to your own messages**: Always check isFromMe.
- **Blocking webhook response**: Process async, respond 200 immediately.
</anti_patterns>
