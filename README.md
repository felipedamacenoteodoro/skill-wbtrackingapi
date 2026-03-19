# WBTrackinAPI Skill

Skill para IAs (Cursor, Claude Code, Windsurf, Cline) construírem integrações com a [WBTrackinAPI](https://wbtrackingapi.com.br) — API WhatsApp com UTM Tracking do Meta Ads.

## Instalação

```bash
npx skills add felipedamacenoteodoro/skill-wbtrackingapi
```

### Manual (Cursor)

Copie a pasta para `.cursor/skills/wbtrackingapi/` no seu projeto.

### Manual (Claude Code)

Copie a pasta para `.claude/skills/wbtrackingapi/` no seu projeto.

## O que a IA aprende

Após instalar, peça para a IA:

- *"Crie um frontend de chat com WhatsApp"*
- *"Crie um webhook para receber mensagens do WhatsApp"*
- *"Crie um bot de atendimento automático"*
- *"Envie uma imagem via WhatsApp para 5511999999999"*
- *"Integre o WhatsApp com meu CRM"*

A IA vai gerar código correto usando a WBTrackinAPI, incluindo:

- Helper library pronta (`lib/wbtrackingapi.js`)
- Autenticação correta (header `token`)
- Formato de telefone correto (sem `+`)
- Webhook receiver com processamento async
- UTM Tracking do Meta Ads
- Upload de arquivos (imagem, PDF, áudio, vídeo)
- Chat React com dark theme

## Estrutura

```
SKILL.md                          # Definição principal + router
references/
  api-endpoints.md                # Todos os endpoints da API
  webhook-payload.md              # Estrutura do webhook + extractors
  helper-library.md               # Módulo JS pronto para copiar
workflows/
  build-chat-frontend.md          # React chat com upload de arquivos
  build-webhook-receiver.md       # Express/FastAPI webhook server
  build-automation.md             # Bot + mensagens em massa + agendamento
```

## Requisitos

Conta ativa na [WBTrackinAPI](https://wbtrackingapi.com.br) com token de instância.
