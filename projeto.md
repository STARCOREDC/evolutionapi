# 📱 Projeto: Multi WhatsApp na VPS com Evolution API

## Visão Geral

Este guia cobre todo o processo para ativar múltiplos números de WhatsApp e hospedá-los em uma VPS usando a Evolution API, sem necessidade de celulares físicos.

---

## 🗺️ Fluxo Completo

```
Números VoIP (com SIP)
        ↓
Ativação no NoxPlayer (PC)
        ↓
Pairing Code → Evolution API (VPS)
        ↓
Números hospedados 24h na VPS
        ↓
Usar via API REST / Painel Web / Webhooks
```

---

## 🛠️ O que você vai precisar

### No PC
- **NoxPlayer** — emulador Android para ativar os números
- **App SIP** — para receber a ligação de verificação do WhatsApp

### Na VPS
- Ubuntu 22.04 LTS
- Node.js 20+
- PM2
- PostgreSQL
- Redis

---

## 📊 Dimensionamento da VPS

| Números | CPU | RAM | Disco |
|---|---|---|---|
| Até 10 | 2 vCPU | 4GB RAM | 50GB SSD |
| 10 a 30 | 4 vCPU | 8GB RAM | 80GB SSD |
| 30 a 60 | 6 vCPU | 16GB RAM | 100GB SSD |
| 60+ | 8 vCPU | 32GB RAM | 150GB SSD |

**Recomendação inicial:** 4 vCPU / 8GB RAM / 80GB SSD

**Sistema Operacional:** Ubuntu 22.04 LTS

**Provedores recomendados:** Hostinger (BR), Contabo, Vultr (Miami), DigitalOcean

---

## 🔵 ETAPA 1 — Ativar os Números no NoxPlayer

### 1.1 Instalar o NoxPlayer

1. Acesse [bignox.com](https://bignox.com) e baixe o NoxPlayer
2. Instale normalmente no seu PC Windows/Mac

### 1.2 Criar múltiplas instâncias

1. Abra o NoxPlayer
2. Clique em **Multi-Drive** (ícone de múltiplas telas)
3. Clique em **Adicionar** para criar quantas instâncias precisar
4. Cada instância = 1 número de WhatsApp

### 1.3 Instalar WhatsApp em cada instância

1. Abra cada instância
2. Acesse a Play Store ou instale o APK do WhatsApp
3. Abra o WhatsApp e coloque o número desejado

### 1.4 Ativar via ligação (SIP)

1. Na tela de verificação, clique em **"Me ligar"** (não SMS)
2. Sua ligação chegará no app SIP do PC
3. Anote o código de 6 dígitos que o robô falar
4. Digite o código no WhatsApp
5. ✅ Número ativado!

> **Dica:** Faça esse processo em todas as instâncias em paralelo para ganhar tempo.

---

## 🟢 ETAPA 2 — Preparar a VPS

### 2.1 Atualizar o sistema

```bash
sudo apt update && sudo apt upgrade -y
```

### 2.2 Instalar Node.js 20

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt install -y nodejs
node -v  # Deve mostrar v20.x.x
```

### 2.3 Instalar PM2

```bash
npm install -g pm2
```

### 2.4 Instalar PostgreSQL

```bash
sudo apt install -y postgresql postgresql-contrib

# Inicia e habilita o serviço
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Cria o banco e usuário
sudo -u postgres psql -c "CREATE USER evolution WITH PASSWORD 'SuaSenhaForte123';"
sudo -u postgres psql -c "CREATE DATABASE evolution OWNER evolution;"
```

### 2.5 Instalar Redis

```bash
sudo apt install -y redis-server

# Inicia e habilita o serviço
sudo systemctl start redis-server
sudo systemctl enable redis-server

# Testa se está funcionando
redis-cli ping  # Deve retornar PONG
```

---

## 🟡 ETAPA 3 — Instalar a Evolution API

### 3.1 Clonar o repositório

```bash
cd /opt
sudo git clone https://github.com/EvolutionAPI/evolution-api
sudo chown -R $USER:$USER /opt/evolution-api
cd /opt/evolution-api
```

### 3.2 Instalar dependências

```bash
npm install
```

### 3.3 Configurar o .env

```bash
cp .env.example .env
nano .env
```

**Configurações principais no .env:**

```env
# Servidor
SERVER_TYPE=http
SERVER_PORT=8080

# Autenticação da API
AUTHENTICATION_TYPE=apikey
AUTHENTICATION_API_KEY=SuaChaveSecretaAqui123

# Banco de dados
DATABASE_ENABLED=true
DATABASE_CONNECTION_URI=postgresql://evolution:SuaSenhaForte123@localhost:5432/evolution

# Redis
CACHE_REDIS_ENABLED=true
CACHE_REDIS_URI=redis://localhost:6379

# Instâncias
DEL_INSTANCE=false
```

> **Importante:** Troque `SuaChaveSecretaAqui123` e `SuaSenhaForte123` por valores seguros.

### 3.4 Fazer o build

```bash
npm run build
```

### 3.5 Subir com PM2

```bash
pm2 start 'npm run start:prod' --name evolution-api
pm2 save
pm2 startup
```

### 3.6 Verificar se está rodando

```bash
pm2 status
# Deve mostrar evolution-api como "online"

# Ver logs em tempo real
pm2 logs evolution-api
```

---

## 🔴 ETAPA 4 — Conectar os Números na Evolution API

### 4.1 Criar uma instância por número

Faça uma requisição POST para a API:

```bash
curl -X POST http://SeuIP:8080/instance/create \
  -H "Content-Type: application/json" \
  -H "apikey: SuaChaveSecretaAqui123" \
  -d '{
    "instanceName": "projeto1",
    "integration": "WHATSAPP-BAILEYS"
  }'
```

### 4.2 Gerar o Pairing Code

```bash
curl -X GET "http://SeuIP:8080/instance/connect/projeto1?pairingCode=true&phoneNumber=5511999999999" \
  -H "apikey: SuaChaveSecretaAqui123"
```

> Substitua `5511999999999` pelo número do WhatsApp com DDI+DDD.

A API vai retornar um **código de 8 dígitos.**

### 4.3 Vincular no WhatsApp (NoxPlayer)

1. Abra o WhatsApp do número no NoxPlayer
2. Vá em **⋮ Menu → Dispositivos conectados**
3. Clique em **Conectar dispositivo**
4. Clique em **Conectar com número de telefone**
5. Digite o código de 8 dígitos gerado
6. ✅ Número conectado na VPS!

> Repita os passos 4.1 a 4.3 para cada número.

---

## 🔄 ETAPA 5 — Uso e Integrações

### 5.1 Enviar mensagem via API

```bash
curl -X POST http://SeuIP:8080/message/sendText/projeto1 \
  -H "Content-Type: application/json" \
  -H "apikey: SuaChaveSecretaAqui123" \
  -d '{
    "number": "5511999999999",
    "text": "Olá! Mensagem enviada pela Evolution API 🚀"
  }'
```

### 5.2 Configurar Webhook (receber mensagens)

```bash
curl -X POST http://SeuIP:8080/webhook/set/projeto1 \
  -H "Content-Type: application/json" \
  -H "apikey: SuaChaveSecretaAqui123" \
  -d '{
    "url": "https://seusite.com/webhook",
    "enabled": true,
    "events": ["MESSAGES_UPSERT", "MESSAGES_UPDATE", "CONNECTION_UPDATE"]
  }'
```

### 5.3 Integrações possíveis

| Ferramenta | Para que serve |
|---|---|
| **N8N** | Automações e fluxos sem código |
| **Chatwoot** | Atendimento humano com inbox |
| **Typebot** | Criação de chatbots visuais |
| **Make/Zapier** | Integração com outros sistemas |

---

## 🔁 Gestão de Reconexões

### Reconexão automática
A Evolution API monitora e tenta reconectar automaticamente. Na maioria dos casos você não precisa fazer nada.

### Verificar status de uma instância

```bash
curl -X GET http://SeuIP:8080/instance/connectionState/projeto1 \
  -H "apikey: SuaChaveSecretaAqui123"
```

### Se precisar reconectar manualmente

1. Gere um novo Pairing Code (passo 4.2)
2. Abra o WhatsApp no NoxPlayer
3. Desconecte o dispositivo antigo em **Dispositivos conectados**
4. Conecte novamente com o novo código

> ⚠️ O NoxPlayer só é necessário nesse momento. Pode ficar fechado no dia a dia.

---

## 🔒 Segurança (Recomendado)

### Configurar firewall

```bash
sudo ufw allow 22      # SSH
sudo ufw allow 8080    # Evolution API (ou use Nginx como proxy)
sudo ufw enable
```

### Usar Nginx como proxy reverso (opcional mas recomendado)

```bash
sudo apt install -y nginx

# Cria o arquivo de configuração
sudo nano /etc/nginx/sites-available/evolution
```

```nginx
server {
    listen 80;
    server_name api.seudominio.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/evolution /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## 📋 Checklist Final

- [ ] NoxPlayer instalado no PC
- [ ] App SIP configurado e recebendo ligações
- [ ] VPS com Ubuntu 22.04 configurada
- [ ] Node.js 20 instalado
- [ ] PM2 instalado
- [ ] PostgreSQL instalado e banco criado
- [ ] Redis instalado e rodando
- [ ] Evolution API clonada e configurada
- [ ] Evolution API rodando via PM2
- [ ] Números ativados no NoxPlayer via SIP
- [ ] Instâncias criadas na Evolution API
- [ ] Pairing Code gerado e vinculado para cada número
- [ ] Firewall configurado
- [ ] Nginx configurado (opcional)

---

## 🆘 Problemas Comuns

| Problema | Solução |
|---|---|
| API não responde | Verificar `pm2 status` e `pm2 logs evolution-api` |
| Pairing Code inválido | Gerar novo código, tem prazo de validade |
| Número desconectou | Verificar status via API e reconectar |
| Redis não conecta | `sudo systemctl restart redis-server` |
| PostgreSQL erro | Verificar credenciais no `.env` |
| WhatsApp banido | Usar o número com moderação no início |

---

## 📚 Links Úteis

- [Evolution API Docs](https://doc.evolution-api.com)
- [Evolution API GitHub](https://github.com/EvolutionAPI/evolution-api)
- [NoxPlayer Download](https://bignox.com)
- [N8N Self-hosted](https://docs.n8n.io/hosting)
- [Chatwoot](https://www.chatwoot.com)

---

*Guia gerado para uso pessoal — sempre consulte a documentação oficial para atualizações.*
