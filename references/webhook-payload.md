<overview>
Structure of webhook payloads received from WBTrackinAPI when messages and events arrive.
Your server receives HTTP POST requests with JSON body.
</overview>

<message_payload>

```json
{
  "type": "Message",
  "instanceName": "instance_name",
  "event": {
    "Info": {
      "ID": "3EB0ABC123DEF456",
      "MessageSource": {
        "Chat": "5511999999999@s.whatsapp.net",
        "Sender": "5511999999999@s.whatsapp.net",
        "IsFromMe": false,
        "IsGroup": false
      },
      "PushName": "João Silva",
      "Timestamp": "2026-03-19T10:30:00Z",
      "Type": "text",
      "Category": ""
    },
    "Message": {
      "conversation": "Plain text message",
      "extendedTextMessage": {
        "text": "Text with formatting or links",
        "contextInfo": {}
      },
      "imageMessage": {
        "caption": "Image caption",
        "mimetype": "image/jpeg",
        "url": "...",
        "fileSha256": "...",
        "fileLength": 12345
      },
      "documentMessage": {
        "caption": "Document description",
        "fileName": "file.pdf",
        "mimetype": "application/pdf",
        "url": "..."
      },
      "audioMessage": {
        "mimetype": "audio/ogg; codecs=opus",
        "ptt": true,
        "url": "..."
      },
      "videoMessage": {
        "caption": "Video caption",
        "mimetype": "video/mp4",
        "url": "..."
      },
      "locationMessage": {
        "degreesLatitude": -23.5505,
        "degreesLongitude": -46.6333,
        "name": "São Paulo"
      },
      "contactMessage": {
        "displayName": "Contact Name",
        "vcard": "BEGIN:VCARD..."
      },
      "stickerMessage": {
        "mimetype": "image/webp"
      }
    }
  },
  "tracking": {
    "utm_source": "FaceAds",
    "utm_campaign": "Summer Campaign 2025",
    "utm_medium": "Lookalike Audience",
    "utm_content": "Video Ad - 30s",
    "utm_term": "-",
    "utm_platform": "MetaAds",
    "ad_id": "120212345678901",
    "ad_name": "Video Ad - 30s",
    "campaign_id": "120210345678901",
    "campaign_name": "Summer Campaign 2025",
    "adset_id": "120211345678901",
    "adset_name": "Lookalike Audience",
  }
}
```

</message_payload>

<extractors>

```javascript
// Extract sender phone number (without @s.whatsapp.net)
function getPhone(webhook) {
  return (webhook.event?.Info?.MessageSource?.Sender || '').replace('@s.whatsapp.net', '');
}

// Extract message text (works for all text message types)
function getText(webhook) {
  const msg = webhook.event?.Message;
  if (!msg) return '';
  return msg.conversation
    || msg.extendedTextMessage?.text
    || msg.imageMessage?.caption
    || msg.videoMessage?.caption
    || msg.documentMessage?.caption
    || '';
}

// Detect message type
function getMessageType(webhook) {
  const msg = webhook.event?.Message;
  if (!msg) return 'unknown';
  if (msg.conversation || msg.extendedTextMessage) return 'text';
  if (msg.imageMessage) return 'image';
  if (msg.audioMessage) return 'audio';
  if (msg.videoMessage) return 'video';
  if (msg.documentMessage) return 'document';
  if (msg.stickerMessage) return 'sticker';
  if (msg.locationMessage) return 'location';
  if (msg.contactMessage) return 'contact';
  return 'unknown';
}

// Check if message was sent by us (ALWAYS check to avoid loops)
function isFromMe(webhook) {
  return webhook.event?.Info?.MessageSource?.IsFromMe || false;
}

// Check if message is from a group
function isGroup(webhook) {
  return webhook.event?.Info?.MessageSource?.IsGroup || false;
}

// Get sender display name
function getPushName(webhook) {
  return webhook.event?.Info?.PushName || '';
}

// Get message ID (for reactions, replies, marking read)
function getMessageId(webhook) {
  return webhook.event?.Info?.ID || '';
}

// Get UTM tracking data (Meta Ads attribution)
function getTracking(webhook) {
  return webhook.tracking || {
    utm_source: 'organico',
    utm_campaign: '-',
    utm_medium: '-',
    utm_content: '-',
    utm_platform: '-'
  };
}

// Check if message came from a paid ad
function isFromAd(webhook) {
  return webhook.tracking?.utm_source === 'FaceAds';
}

// Get group JID (for group messages)
function getGroupJid(webhook) {
  if (!isGroup(webhook)) return null;
  return webhook.event?.Info?.MessageSource?.Chat || null;
}
```

</extractors>

<tracking_classifications>

| utm_source | utm_platform | Meaning |
|---|---|---|
| FaceAds | MetaAds | Message from a Meta Ad (Click-to-WhatsApp) |
| organico | facebook | Organic click from Facebook |
| organico | instagram | Organic click from Instagram |
| Indicacao | - | Message from a shared contact card |
| organico | - | Direct message, no attribution |

The tracking object is automatically populated by WBTrackinAPI when the message contains
O tracking é resolvido automaticamente pela WBTrackinAPI quando configurado o Meta Ads Token.

</tracking_classifications>

<event_types>

| type | Description |
|---|---|
| Message | New message received |
| ReadReceipt | Message was read (state: Read, ReadSelf, Delivered) |
| Presence | User online/offline |
| ChatPresence | User typing/recording in a specific chat |
| Connected | WhatsApp session connected |
| Disconnected | WhatsApp session disconnected |
| LoggedOut | Session logged out (needs new QR) |
| GroupInfo | Group metadata changed |

</event_types>
