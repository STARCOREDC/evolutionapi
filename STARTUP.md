# 🚀 Evolution API — Guia de Startup

> Documento de referência rápida da instalação rodando nesta VPS.
> Última atualização: 2026-04-27

---

## 📍 O que está rodando

| Item | Valor |
|---|---|
| **URL pública** | https://evo.starcoredc.com |
| **Painel web** | https://evo.starcoredc.com/manager/login |
| **Versão** | Evolution API v2.3.7 |
| **Diretório** | `/opt/evolutionapi/evolution-api` |
| **Process manager** | PM2 (auto-start no boot) |
| **Banco** | PostgreSQL 14 (local) |
| **Cache** | Redis (local) |
| **Porta interna** | 8080 |
| **SSL** | Gerenciado pelo painel externo (Let's Encrypt) |
| **VPS** | 179.42.75.250 (público) / 10.0.1.155 (privado) |

---

## 🔑 Credenciais

> ⚠️ Arquivo completo (com permissão 600): `/root/evolution-credentials.txt`
>
> ```bash
> cat /root/evolution-credentials.txt
> ```

**API Key Global (header `apikey:`):**
```
b3d1b010e86872c66320f47bc0656df434f26eb4b0750767a2b3105875eb9b8f
```

---

## ⚡ Login no painel

1. Acessa https://evo.starcoredc.com/manager/login
2. Cola a API Key acima
3. Pronto

---

## 🟢 Comandos do dia-a-dia

```bash
# Status
pm2 status

# Logs em tempo real
pm2 logs evolution-api

# Logs limitados
pm2 logs evolution-api --lines 100

# Reiniciar
pm2 restart evolution-api

# Parar / Iniciar
pm2 stop evolution-api
pm2 start evolution-api

# Monitor de CPU/RAM
pm2 monit
```

---

## 📱 Criar uma instância (resumo)

### Pelo painel (recomendado)

1. Login no `/manager/login`
2. **Create Instance** → name: `projeto1` → integration: **`WHATSAPP-BAILEYS`** → Save
3. Clica na instância → **Connect**
4. Escolhe **Pairing Code** → digita o número com DDI (`5511999999999`)
5. Painel mostra os 8 dígitos
6. WhatsApp no NoxPlayer → ⋮ → **Dispositivos conectados** → **Conectar com número** → cola

### Via API (curl)

```bash
# Criar instância
curl -X POST https://evo.starcoredc.com/instance/create \
  -H "Content-Type: application/json" \
  -H "apikey: b3d1b010e86872c66320f47bc0656df434f26eb4b0750767a2b3105875eb9b8f" \
  -d '{"instanceName":"projeto1","integration":"WHATSAPP-BAILEYS"}'

# Gerar pairing code
curl "https://evo.starcoredc.com/instance/connect/projeto1?pairingCode=true&phoneNumber=5511999999999" \
  -H "apikey: b3d1b010e86872c66320f47bc0656df434f26eb4b0750767a2b3105875eb9b8f"
```

---

## 💬 Enviar mensagens (exemplos)

```bash
# Texto
curl -X POST https://evo.starcoredc.com/message/sendText/projeto1 \
  -H "Content-Type: application/json" \
  -H "apikey: b3d1b010e86872c66320f47bc0656df434f26eb4b0750767a2b3105875eb9b8f" \
  -d '{"number":"5511999999999","text":"Olá, mensagem da API"}'

# Listar instâncias
curl https://evo.starcoredc.com/instance/fetchInstances \
  -H "apikey: b3d1b010e86872c66320f47bc0656df434f26eb4b0750767a2b3105875eb9b8f"

# Ver status de conexão
curl https://evo.starcoredc.com/instance/connectionState/projeto1 \
  -H "apikey: b3d1b010e86872c66320f47bc0656df434f26eb4b0750767a2b3105875eb9b8f"
```

Documentação completa de endpoints: https://doc.evolution-api.com

---

## 🔄 Atualizar a Evolution API

Quando sair uma versão nova:

```bash
cd /opt/evolutionapi/evolution-api
git pull
npm install
npm run db:deploy
npm run build
pm2 restart evolution-api
pm2 logs evolution-api --lines 30 --nostream  # validar que subiu sem erro
```

---

## 🆘 Troubleshooting

### API não responde

```bash
pm2 status                          # tá online?
pm2 logs evolution-api --lines 50   # ver erros
sudo systemctl status postgresql    # banco ok?
sudo systemctl status redis-server  # cache ok?
```

### Reiniciar tudo do zero

```bash
sudo systemctl restart postgresql
sudo systemctl restart redis-server
pm2 restart evolution-api
```

### Banco não conecta

Verifica `DATABASE_CONNECTION_URI` no `.env`:

```bash
grep DATABASE_CONNECTION_URI /opt/evolutionapi/evolution-api/.env
```

E testa direto:

```bash
sudo -u postgres psql -c "\l" | grep evolution
```

### Instância caiu / desconectou

1. No painel `/manager`, clica na instância
2. Se status = `close`, clica **Connect** e gera novo pairing code
3. No NoxPlayer, abre o WhatsApp do número → desconecta o dispositivo antigo em **Dispositivos conectados** → conecta de novo com o código novo

> ⚠️ **Mantenha o número logado no NoxPlayer abrindo o WhatsApp pelo menos 1x a cada ~14 dias** pra Meta não suspender por inatividade.

---

## 🔥 Firewall (UFW)

```bash
ufw status numbered
```

Regras atuais:
- 22/tcp (SSH) — qualquer origem
- 80/tcp (HTTP) — qualquer origem
- 443/tcp (HTTPS) — qualquer origem
- 8080/tcp (Evolution) — só rede privada (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)

---

## ⚠️ Cuidados com o número (anti-ban)

1. **Aquecimento (primeiros 3-7 dias):**
   - Mande só pra contatos conhecidos (5-15 mensagens/dia)
   - Receba mensagens de volta (conversa de mão dupla)
   - Adicione foto, nome, descrição no perfil
   - Entra em alguns grupos, manda áudio, foto

2. **Disparo em massa:**
   - **Nunca** comece direto com lista grande
   - Intervalo aleatório entre mensagens (5-30 segundos)
   - Não copie/cole o mesmo texto idêntico
   - Use variações: `{{nome}}`, emojis diferentes, ordem alterada

3. **Manutenção do número:**
   - Abre o WhatsApp normal pelo menos 1x a cada 2 semanas
   - Se receber muitos blocks, pausa o número 3-7 dias

---

## 🔌 Integrações disponíveis (built-in)

A Evolution API já vem pronta para conectar com:

| Tipo | Ferramentas |
|---|---|
| **Chatbot** | OpenAI, Dify, Typebot, Flowise, n8n, EvoAI |
| **Atendimento** | Chatwoot |
| **Filas/Eventos** | RabbitMQ, SQS, NATS, Kafka, Pusher, WebSocket |
| **Storage** | S3, MinIO |
| **Webhook** | Configurável por instância (qualquer URL HTTP) |

Pra ativar: edite `.env` e ative a variável correspondente (`OPENAI_ENABLED=true`, `CHATWOOT_ENABLED=true`, etc.) → `pm2 restart evolution-api`.

---

## 📂 Arquivos importantes

```
/opt/evolutionapi/
├── projeto.md                        # guia original do projeto
├── STARTUP.md                        # este arquivo
└── evolution-api/                    # código-fonte
    ├── .env                          # configurações (chmod 600)
    ├── dist/                         # build compilado
    ├── prisma/                       # migrations
    └── src/                          # código TypeScript

/root/
└── evolution-credentials.txt         # credenciais e exemplos (chmod 600)

/etc/nginx/sites-available/evolution  # config nginx (DESATIVADA — painel externo cuida)
```

---

## 🛣️ Roadmap pessoal — quando voltar

- [ ] Aquecer número conectado (3-7 dias)
- [ ] Conectar números adicionais (NoxPlayer + pairing code)
- [ ] Integrar com painel próprio de hospedagem (cobranças automáticas)
- [ ] Configurar webhook pra receber mensagens
- [ ] Avaliar bot/IA (Typebot ou OpenAI direto)
- [ ] Plano SaaS futuro: billing, multi-tenant, white-label

---

## 📚 Links

- Evolution API Docs: https://doc.evolution-api.com
- Evolution API GitHub: https://github.com/EvolutionAPI/evolution-api
- Postman Collection: https://doc.evolution-api.com (seção Postman)
- Discord oficial: https://evolution-api.com/discord
