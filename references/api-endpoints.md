<overview>
Complete API reference for WBTrackinAPI. All endpoints require the `token` header (lowercase).
Base URL comes from environment variable `WBTRACKINGAPI_URL`.
</overview>

<endpoints>

## Sending Messages

<endpoint name="Send Text">
POST /chat/send/text
Body: {"Phone": "5511999999999", "Body": "Message text"}
Response: {"success": true, "data": {"Id": "3EB0..."}}
</endpoint>

<endpoint name="Send Image">
POST /chat/send/image
Body: {"Phone": "5511999999999", "Image": "data:image/jpeg;base64,...", "Caption": "Optional caption"}
Notes: Supports JPEG and PNG. Max ~16MB after base64 encoding.
</endpoint>

<endpoint name="Send Document">
POST /chat/send/document
Body: {"Phone": "5511999999999", "Document": "data:application/pdf;base64,...", "FileName": "contract.pdf", "Caption": "Optional description"}
Notes: Any file type. FileName must include extension.
</endpoint>

<endpoint name="Send Audio">
POST /chat/send/audio
Body: {"Phone": "5511999999999", "Audio": "data:audio/ogg;base64,...", "Ptt": true}
Notes: Ptt=true sends as voice message (green bubble). Ptt=false sends as audio file.
</endpoint>

<endpoint name="Send Video">
POST /chat/send/video
Body: {"Phone": "5511999999999", "Video": "data:video/mp4;base64,...", "Caption": "Optional caption"}
Notes: MP4 with H.264 codec recommended.
</endpoint>

<endpoint name="Send Location">
POST /chat/send/location
Body: {"Phone": "5511999999999", "Latitude": -23.5505, "Longitude": -46.6333, "Name": "Location Name"}
</endpoint>

<endpoint name="Send Contact">
POST /chat/send/contact
Body: {"Phone": "5511999999999", "ContactName": "João Silva", "ContactPhone": "5511888888888"}
</endpoint>

<endpoint name="Send Buttons">
POST /chat/send/buttons
Body: {"Phone": "5511999999999", "Title": "Choose an option", "Buttons": [{"ButtonId": "1", "ButtonText": "Option 1"}, {"ButtonId": "2", "ButtonText": "Option 2"}]}
Notes: Maximum 3 buttons.
</endpoint>

<endpoint name="Send List">
POST /chat/send/list
Body: {"Phone": "5511999999999", "Title": "Menu", "Description": "Select an item", "ButtonText": "View options", "Sections": [{"Title": "Section Name", "Rows": [{"RowId": "1", "Title": "Item 1", "Description": "Description"}]}]}
</endpoint>

## Chat Actions

<endpoint name="Mark as Read">
POST /chat/markread
Body: {"Id": ["message_id_1", "message_id_2"], "Chat": "5511999999999@s.whatsapp.net"}
</endpoint>

<endpoint name="Set Typing/Recording Presence">
POST /chat/presence
Body: {"Phone": "5511999999999", "State": "composing"}
Notes: States are "composing" (typing), "paused" (stopped typing), "recording" (recording audio).
</endpoint>

<endpoint name="React to Message">
POST /chat/react
Body: {"Phone": "5511999999999", "Body": "👍", "Id": "message_id"}
</endpoint>

<endpoint name="Delete Message">
POST /chat/delete
Body: {"Phone": "5511999999999", "Id": "message_id"}
Notes: Can only delete messages sent by you.
</endpoint>

## Download Received Media

<endpoint name="Download Image/Video/Audio/Document">
POST /chat/downloadimage
POST /chat/downloadvideo
POST /chat/downloadaudio
POST /chat/downloaddocument
Body: {"Phone": "5511999999999", "Id": "message_id"}
Response: {"success": true, "data": {"data": "base64_content", "mimetype": "image/jpeg"}}
</endpoint>

## Session

<endpoint name="Check Status">
GET /session/status
Response: {"success": true, "data": {"Connected": true, "LoggedIn": true}}
</endpoint>

<endpoint name="Connect">
POST /session/connect
Body: {"Subscribe": ["All"], "Immediate": true}
</endpoint>

<endpoint name="Get QR Code">
GET /session/qr
Response: {"success": true, "data": {"QRCode": "data:image/png;base64,..."}}
</endpoint>

<endpoint name="Pair by Phone">
POST /session/pairphone
Body: {"Phone": "5511999999999"}
Response: {"success": true, "data": {"LinkingCode": "XXXX-XXXX"}}
</endpoint>

<endpoint name="Logout">
POST /session/logout
</endpoint>

## Users/Contacts

<endpoint name="Check if Number has WhatsApp">
POST /user/check
Body: {"Phone": ["5511999999999", "5511888888888"]}
Response: {"success": true, "data": [{"Query": "5511999999999", "IsInWhatsapp": true, "JID": "5511999999999@s.whatsapp.net"}]}
</endpoint>

<endpoint name="Get Contacts">
GET /user/contacts
Response: {"success": true, "data": {"5511999999999@s.whatsapp.net": {"FullName": "João", "PushName": "João"}}}
</endpoint>

<endpoint name="Get Avatar">
POST /user/avatar
Body: {"Phone": "5511999999999", "Preview": false}
Response: {"success": true, "data": {"url": "https://..."}}
</endpoint>

## Webhook Configuration

<endpoint name="Set Webhook">
POST /webhook
Body: {"WebhookURL": "https://yourserver.com/webhook", "Events": ["Message"]}
Notes: Para receber mensagens do WhatsApp, use apenas o evento "Message". Os demais eventos (ReadReceipt, Presence, ChatPresence, Connected, etc.) são opcionais e geralmente não são necessários para integrações comuns. Use "All" somente se precisar de todos os eventos.
</endpoint>

<endpoint name="Get Webhook">
GET /webhook
Response: {"code": 200, "data": {"webhook": "https://...", "subscribe": ["All"]}}
</endpoint>

<endpoint name="Delete Webhook">
DELETE /webhook
</endpoint>

## Meta Ads Tracking

<endpoint name="Configurar Token Meta Ads">
POST /session/meta-ads/config
Header: token: {TOKEN}
Body: {"meta_ads_token": "EAAxxxxxxxxxxxxxxxx"}
Response: {"success": true, "message": "Meta Ads token saved successfully"}
Notes: Salva o token de acesso do Meta Ads (criptografado). Precisa da permissão ads_read. Quando configurado, o webhook de mensagens inclui automaticamente o objeto tracking com dados de campanha.
</endpoint>

<endpoint name="Verificar Config Meta Ads">
GET /session/meta-ads/config
Header: token: {TOKEN}
Response: {"success": true, "has_meta_token": true}
Notes: Retorna se o token está configurado. O token nunca é retornado por segurança.
</endpoint>

<endpoint name="Remover Config Meta Ads">
DELETE /session/meta-ads/config
Header: token: {TOKEN}
Response: {"success": true, "message": "Meta Ads configuration deleted"}
</endpoint>

<endpoint name="Testar Token Meta Ads">
POST /session/meta-ads/test
Header: token: {TOKEN}
Body: {"token": "EAAxxxxxxxxxxxxxxxx"}
Response: {"success": true, "is_valid": true, "scopes": ["ads_read", "ads_management"]}
Notes: Valida um token Meta Ads antes de salvar. Verifica se é válido e retorna as permissões.
</endpoint>

## UTM Links

<endpoint name="Criar UTM Link">
POST /admin/utm-links
Header: Authorization: {ADMIN_TOKEN}
Body: {"name": "Bio Instagram", "phone": "5511999999999", "message": "Olá! Vi no Instagram", "utm_source": "instagram", "utm_medium": "bio_link", "utm_campaign": "captacao_leads", "utm_content": "link_bio"}
Response: {"success": true, "data": {"id": "abc", "code": "627493bd", "short_url": "https://wbtrackingapi.com.br/r/627493bd", "wa_url": "https://wa.me/5511999999999?text=..."}}
Notes: Cria um link rastreável. O short_url redireciona para wa.me com hash invisível na mensagem. Quando o usuário envia a mensagem, o tracking é preenchido automaticamente com os UTM params. O hash usa Zero-Width Unicode characters — invisível ao usuário.
</endpoint>

<endpoint name="Listar UTM Links">
GET /admin/utm-links
Header: Authorization: {ADMIN_TOKEN}
Response: {"success": true, "data": [{"id": "abc", "code": "627493bd", "name": "Bio Instagram", "phone": "5511999999999", "short_url": "https://...", "utm_source": "instagram", "clicks": 42}]}
</endpoint>

<endpoint name="Excluir UTM Link">
DELETE /admin/utm-links/{id}
Header: Authorization: {ADMIN_TOKEN}
Response: {"success": true, "message": "Link deleted"}
</endpoint>

<endpoint name="Redirect (público)">
GET /r/{code}
Notes: Endpoint público. Registra o clique, incrementa contador, e redireciona para wa.me com hash invisível na mensagem pré-preenchida. Não requer autenticação.
</endpoint>

## Groups

<endpoint name="List Groups">
GET /group/list
</endpoint>

<endpoint name="Create Group">
POST /group/create
Body: {"Name": "Group Name", "Participants": ["5511999999999@s.whatsapp.net"]}
</endpoint>

</endpoints>

<phone_format>
Always use country code + area code + number without any prefix:
- Brazil: 5511999999999 (55 = country, 11 = area, 999999999 = number)
- Argentina: 5491155551234
- USA: 12125551234
- Never use: +5511999999999, 55-11-99999-9999, (11) 99999-9999
- For groups: use the full JID like 120363xxx@g.us
- For @s.whatsapp.net suffix: only add when the endpoint specifically requires it (markread, check)
</phone_format>

<file_conversion>

```javascript
// File to base64 data URI (Node.js)
const fs = require('fs');
const mime = require('mime-types');
function fileToBase64(filePath) {
  const data = fs.readFileSync(filePath);
  const mimeType = mime.lookup(filePath) || 'application/octet-stream';
  return `data:${mimeType};base64,${data.toString('base64')}`;
}

// File to base64 data URI (Browser)
function fileToBase64Browser(file) {
  return new Promise((resolve) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.readAsDataURL(file);
  });
}
```

</file_conversion>
