<div align="center">

# 📱 Evolution API — STARCOREDC

Instância auto-hospedada da **Evolution API v2** para envio e recebimento de mensagens WhatsApp via API REST, com suporte a múltiplas instâncias, webhooks e integrações com chatbots e ferramentas de automação.

[![Evolution API](https://img.shields.io/badge/Evolution_API-v2.3.7-blue?style=for-the-badge)](https://github.com/EvolutionAPI/evolution-api)
[![Node.js](https://img.shields.io/badge/Node.js-20+-green?style=for-the-badge&logo=node.js)](https://nodejs.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-14-blue?style=for-the-badge&logo=postgresql)](https://postgresql.org)
[![Redis](https://img.shields.io/badge/Redis-Cache-red?style=for-the-badge&logo=redis)](https://redis.io)
[![PM2](https://img.shields.io/badge/PM2-Process_Manager-2B037A?style=for-the-badge)](https://pm2.keymetrics.io)

</div>

---

## 🌐 Acesso

| Recurso | URL |
|---|---|
| **API REST** | `https://evo.starcoredc.com` |
| **Painel Web** | `https://evo.starcoredc.com/manager/login` |
| **Documentação oficial** | [doc.evolution-api.com](https://doc.evolution-api.com) |

---

## 🏗️ Stack

| Componente | Tecnologia |
|---|---|
| Core | Evolution API v2.3.7 |
| Runtime | Node.js 20 + TypeScript |
| Banco de dados | PostgreSQL 14 |
| Cache | Redis |
| Process manager | PM2 (auto-start no boot) |
| Proxy reverso | Nginx + SSL |

---

## 🚀 Funcionalidades

- ✅ Múltiplas instâncias WhatsApp simultâneas
- ✅ Conexão via **Pairing Code** (sem QR Code físico)
- ✅ Envio de texto, mídia, áudio, documentos, localização
- ✅ Recebimento via **Webhook** por instância
- ✅ Grupos: criar, gerenciar, enviar mensagens
- ✅ Integração nativa com **OpenAI, Typebot, n8n, Chatwoot, Dify, Flowise**
- ✅ Storage de mídia via **S3 / MinIO**
- ✅ Filas com **RabbitMQ, Kafka, NATS, SQS**

---

## ⚡ Uso rápido

### Criar instância

```bash
curl -X POST https://evo.starcoredc.com/instance/create \
  -H "Content-Type: application/json" \
  -H "apikey: SUA_API_KEY" \
  -d '{"instanceName":"minha-instancia","integration":"WHATSAPP-BAILEYS"}'
```

### Conectar número via Pairing Code

```bash
curl "https://evo.starcoredc.com/instance/connect/minha-instancia?pairingCode=true&phoneNumber=5511999999999" \
  -H "apikey: SUA_API_KEY"
```

### Enviar mensagem

```bash
curl -X POST https://evo.starcoredc.com/message/sendText/minha-instancia \
  -H "Content-Type: application/json" \
  -H "apikey: SUA_API_KEY" \
  -d '{"number":"5511999999999","text":"Ola! 👋"}'
```

### Verificar status da instância

```bash
curl https://evo.starcoredc.com/instance/connectionState/minha-instancia \
  -H "apikey: SUA_API_KEY"
```

---

## 🛠️ Gerenciamento no servidor

```bash
# Status dos processos
pm2 status

# Logs em tempo real
pm2 logs evolution-api

# Reiniciar
pm2 restart evolution-api

# Monitor de CPU/RAM
pm2 monit
```

---

## 🔄 Atualizar versão

```bash
cd /opt/evolutionapi/evolution-api
git pull
npm install
npm run db:deploy
npm run build
pm2 restart evolution-api
```

---

## 🔌 Integrações disponíveis

| Categoria | Ferramentas |
|---|---|
| **Chatbot / IA** | OpenAI, Dify, Typebot, Flowise, n8n, EvoAI |
| **Atendimento** | Chatwoot |
| **Filas / Eventos** | RabbitMQ, SQS, NATS, Kafka, Pusher, WebSocket |
| **Storage** | S3, MinIO |
| **Webhook** | Configuravel por instância |

---

## 🆘 Troubleshooting rápido

| Problema | Solução |
|---|---|
| API não responde | `pm2 status` e `pm2 logs evolution-api` |
| Pairing Code inválido | Gerar novo código (tem prazo de validade) |
| Instância desconectou | Reconectar via painel ou API |
| Redis não conecta | `sudo systemctl restart redis-server` |
| PostgreSQL erro | Verificar `DATABASE_CONNECTION_URI` no `.env` |

---

## 📚 Referências

- [Evolution API Docs](https://doc.evolution-api.com)
- [Evolution API GitHub](https://github.com/EvolutionAPI/evolution-api)

---

<div align="center">
  <sub>Mantido por <a href="https://github.com/STARCOREDC">STARCOREDC</a> · Hospedado em infraestrutura própria</sub>
</div>
