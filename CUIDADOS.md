# 🛡️ Cuidados — Evolution API + WhatsApp

> Guia de boas práticas pra evitar bans, perdas de número, invasões e prejuízo.
> Última atualização: 2026-04-27

---

## 🟥 Cuidados com o número de WhatsApp (anti-ban)

### 1. Aquecimento OBRIGATÓRIO de número novo

Conta nunca usada na API tem ~80% de chance de ban se já começar mandando em massa.

**Primeiros 7 dias:**

| Dia | Mensagens/dia | Tipo |
|---|---|---|
| 1-2 | 5 a 15 | Só pra contatos conhecidos. **Receba respostas.** |
| 3-4 | 20 a 40 | Aumenta um pouco. Misture texto, áudio, foto. |
| 5-7 | 50 a 100 | Pode começar com desconhecidos. Aleatorize. |
| 8+ | até 500/dia | Mantém variação humana. |

> **Receber é tão importante quanto enviar.** Conta que só envia = robô óbvio pra Meta.

### 2. Perfil COMPLETO antes de qualquer disparo

- ✅ **Foto de perfil** (nada genérico tipo logo simples — algo com cara de pessoa/empresa real)
- ✅ **Nome** legível (não use só números ou caracteres estranhos)
- ✅ **Descrição** preenchida ("Sobre")
- ✅ Pelo menos 5-10 conversas reais antes do primeiro disparo

### 3. NUNCA faça isso

- ❌ Mandar a **mesma mensagem idêntica** pra 100+ pessoas (Meta detecta na hora)
- ❌ Mandar pra números que **nunca te mandaram mensagem antes** sem aquecimento
- ❌ **Spammar grupos** com link/promo
- ❌ Mandar **muito rápido** (sem intervalo entre mensagens)
- ❌ Usar **gírias de spam** ("PROMOÇÃO IMPERDÍVEL", "GANHE DINHEIRO RÁPIDO")
- ❌ Adicionar muita gente em grupos sem permissão

### 4. Disparo seguro (quando for fazer)

```
Intervalo entre mensagens:  5 a 30 segundos (aleatório)
Pausa a cada 50 envios:     2 a 5 minutos
Pausa noturna:              22h às 8h (ninguém manda promo de madrugada)
Variação de texto:          {{nome}}, emojis diferentes, sinônimos, ordem trocada
Limite diário:              número novo: 200 / aquecido: 500-1000
```

### 5. Sinais de que o número vai ser banido

- 🟡 Várias pessoas **bloqueando** seu número em sequência
- 🟡 Mensagens não chegando (1 cinza só, não 2)
- 🟡 Lentidão na entrega
- 🟡 WhatsApp pedindo **re-verificação** com frequência
- 🔴 Aviso "**Você está usando uma versão não-oficial**" → desconecte e reduza disparo IMEDIATAMENTE

### 6. Manutenção mensal do número

- **Abra o WhatsApp normal** (não-API) no NoxPlayer pelo menos **1x a cada 14 dias** — Meta marca como inativo senão
- Mantenha o número com **chip ativo** (no caso de VoIP, mantenha o crédito ou plano)
- Se receber muitos blocks/reports → **pausa de 3-7 dias** sem disparo, só conversa normal

---

## 🔐 Segurança da VPS

### 1. API Key — proteja como senha de banco

A `AUTHENTICATION_API_KEY` é equivalente a **senha root da sua API**. Quem tiver ela:
- Cria/deleta instâncias
- Lê todas as mensagens
- Envia mensagens em seu nome
- Vê seus contatos e grupos

**Onde NUNCA colocar:**
- ❌ Repositório Git público
- ❌ Frontend/JavaScript de cliente (qualquer um vê)
- ❌ Mensagens em grupo, prints, logs públicos
- ❌ Cliente final (.env do app web do cliente)

**Onde SIM colocar:**
- ✅ Variáveis de ambiente do **servidor** (backend)
- ✅ Cofre de segredos (HashiCorp Vault, AWS Secrets Manager, etc.)
- ✅ `.env` com `chmod 600` (já está feito)

**Se vazar:**

```bash
# 1. Gera nova chave
NEW_KEY=$(openssl rand -hex 32)

# 2. Substitui no .env
sed -i "s/^AUTHENTICATION_API_KEY=.*/AUTHENTICATION_API_KEY=$NEW_KEY/" \
  /opt/evolutionapi/evolution-api/.env

# 3. Reinicia
pm2 restart evolution-api

# 4. Atualiza a chave em todos os clientes/integrações que usam
echo "Nova chave: $NEW_KEY"
```

### 2. SSH — proteja a entrada da VPS

- ✅ Use **chave SSH** (nunca senha)
- ✅ Desabilite **login root direto** (use sudo)
- ✅ Mude a porta padrão 22 pra outra (ex.: 2222)
- ✅ Configure **fail2ban** pra bloquear tentativas de força bruta
- ❌ Nunca compartilhe sua chave privada `id_rsa`

```bash
# Instalar fail2ban (recomendado)
apt install -y fail2ban
systemctl enable --now fail2ban
```

### 3. Atualizações de segurança

```bash
# Mensal — atualiza pacotes do Ubuntu
apt update && apt upgrade -y

# A cada 3 meses — atualiza Evolution API
cd /opt/evolutionapi/evolution-api
git pull && npm install && npm run db:deploy && npm run build && pm2 restart evolution-api
```

### 4. Firewall — revise periodicamente

```bash
ufw status numbered
```

Só deve ter aberto:
- 22 (SSH)
- 80 (HTTP) — pro Let's Encrypt validar
- 443 (HTTPS)
- 8080 — **só rede privada interna** (não pode estar `Anywhere`)

Se ver `8080/tcp ALLOW Anywhere`, **fecha agora:**

```bash
ufw delete allow 8080
```

### 5. Logs — monitore de vez em quando

```bash
# Tentativas de login SSH falhadas
journalctl -u ssh --since "24 hours ago" | grep "Failed"

# Logs da API
pm2 logs evolution-api --lines 200 --nostream

# Espaço em disco (não pode lotar!)
df -h /
```

---

## 💾 Backup

### Por que fazer

Se a VPS for comprometida, deletada ou der pau de hardware, **você perde tudo:**
- Sessões dos números (vai ter que reconectar TODOS)
- Histórico de mensagens
- Configurações de webhooks/integrações
- Banco de contatos

### O que fazer backup

| Item | Caminho | Como |
|---|---|---|
| **Banco PostgreSQL** | DB `evolution` | `pg_dump` |
| **`.env`** | `/opt/evolutionapi/evolution-api/.env` | copiar arquivo |
| **Sessões Baileys** | `/opt/evolutionapi/evolution-api/instances/` | copiar pasta |
| **Credenciais** | `/root/evolution-credentials.txt` | copiar arquivo |

### Script de backup diário

Cria `/root/backup-evolution.sh`:

```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M)
DEST=/root/backups
mkdir -p $DEST

# Banco
sudo -u postgres pg_dump evolution | gzip > $DEST/db_$DATE.sql.gz

# .env e sessões
tar czf $DEST/files_$DATE.tar.gz \
  /opt/evolutionapi/evolution-api/.env \
  /opt/evolutionapi/evolution-api/instances/ 2>/dev/null

# Mantém só últimos 7 backups
ls -1t $DEST/db_*.sql.gz | tail -n +8 | xargs -r rm
ls -1t $DEST/files_*.tar.gz | tail -n +8 | xargs -r rm

echo "Backup OK: $DATE"
```

```bash
chmod +x /root/backup-evolution.sh

# Adiciona no cron — diário às 3h da manhã
(crontab -l 2>/dev/null; echo "0 3 * * * /root/backup-evolution.sh >> /var/log/backup-evolution.log 2>&1") | crontab -
```

### Backup off-site (recomendado)

Backup local não protege se a VPS for perdida. Sincronize pra:
- **Backblaze B2** (~$5/TB/mês, barato)
- **Wasabi** (~$6/TB/mês)
- **AWS S3** (mais caro, mais robusto)
- **Google Drive / Dropbox** pessoal (até 15GB grátis)
- **Outra VPS** via `rsync` ou `restic`

---

## 📊 Monitoramento

### Verificar saúde da API

```bash
# Tá viva?
curl -s https://evo.starcoredc.com/ | head

# Memória / CPU dos processos
pm2 monit

# Recursos da VPS
free -h          # RAM
df -h /          # disco
top              # CPU em tempo real
htop             # melhor que top (apt install htop)
```

### Sinais de problema

| Sinal | Causa provável | Ação |
|---|---|---|
| API não responde | App caiu | `pm2 restart evolution-api` + ver logs |
| Mensagens lentas | RAM/CPU lotado | `pm2 monit` ver consumo |
| Disco lotado | Logs ou sessões grandes | `du -sh /opt/evolutionapi/evolution-api/*` |
| Vários números desconectando | Problema de rede ou Meta | Aguardar 30min, reconectar |
| Erro de banco | Postgres caiu | `systemctl restart postgresql` |

### Alerta automático (opcional)

Configure um cron pra te avisar no WhatsApp se a API cair:

```bash
# /root/health-check.sh
#!/bin/bash
if ! curl -sf https://evo.starcoredc.com/ > /dev/null; then
  curl -X POST https://evo.starcoredc.com/message/sendText/projeto1 \
    -H "apikey: SUA_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{"number":"5511SEUWHATSAPP","text":"⚠️ Evolution API caiu!"}' || true
fi
```

(esse exemplo só funciona enquanto o "projeto1" estiver online — pra alerta de verdade use Uptime Kuma ou Better Uptime)

---

## 🚫 O que NÃO fazer (resumo)

1. ❌ **Não compartilhe** sua API Key com ninguém que você não confia 100%
2. ❌ **Não exponha** a porta 8080 pra internet pública (`ufw allow 8080` sem origem)
3. ❌ **Não comece disparo** com número novo sem aquecimento
4. ❌ **Não mande** a mesma mensagem idêntica pra muita gente
5. ❌ **Não esqueça** de fazer backup do banco e das sessões
6. ❌ **Não atualize** a Evolution API direto em produção sem testar antes
7. ❌ **Não rode** `npm audit fix --force` (quebra a aplicação)
8. ❌ **Não delete** a pasta `/opt/evolutionapi/evolution-api/instances/` (perde TODAS as sessões)
9. ❌ **Não desligue** o PostgreSQL/Redis enquanto a API estiver rodando
10. ❌ **Não rode `git pull`** sem fazer backup antes (migrations podem ser destrutivas)

---

## ✅ Checklist mensal de saúde

- [ ] API respondendo (`curl https://evo.starcoredc.com/`)
- [ ] PM2 com auto-restart funcionando (`pm2 status`)
- [ ] Backup rodando (`ls -la /root/backups/`)
- [ ] Disco com pelo menos 20% livre (`df -h /`)
- [ ] RAM com pelo menos 30% livre (`free -h`)
- [ ] Logs sem erros recorrentes (`pm2 logs evolution-api --lines 500 --nostream | grep -i error`)
- [ ] Firewall ainda só com portas necessárias (`ufw status`)
- [ ] Números abertos manualmente no NoxPlayer pelo menos 1x (cada um)
- [ ] Sistema operacional atualizado (`apt update && apt upgrade`)

---

## 🆘 Em caso de emergência

### "Meu número foi banido"

- **Não dá pra reverter** quando a Meta bana definitivo
- Tente recurso direto pelo WhatsApp do celular (botão "Solicitar revisão")
- Lições: aquecer mais e disparar menos no próximo número

### "Perdi acesso à VPS"

1. Acesse pelo painel do provedor (Hostinger/Contabo/etc.) — console web
2. Reset a senha root pelo painel
3. Restaure das credenciais salvas em `/root/evolution-credentials.txt`

### "API caiu e não sobe"

```bash
# 1. Ver erro exato
pm2 logs evolution-api --lines 100 --nostream

# 2. Tentar restart limpo
pm2 delete evolution-api
cd /opt/evolutionapi/evolution-api
pm2 start "npm run start:prod" --name evolution-api
pm2 save

# 3. Se falhar, tentar reconstruir
npm run build
pm2 restart evolution-api

# 4. Último caso: restaurar do backup
gunzip -c /root/backups/db_AAAAMMDD_HHMM.sql.gz | sudo -u postgres psql evolution
```

### "Esqueci a API Key"

```bash
grep AUTHENTICATION_API_KEY /opt/evolutionapi/evolution-api/.env
# OU
cat /root/evolution-credentials.txt
```

---

## 📞 Onde buscar ajuda

- 📚 Docs oficiais: https://doc.evolution-api.com
- 🐙 GitHub Issues: https://github.com/EvolutionAPI/evolution-api/issues
- 💬 Discord: https://evolution-api.com/discord
- 📧 Email do mantenedor: contato@evolution-api.com

---

**Lembre-se:** prevenção é 100x mais barata que recuperação. Backup + aquecimento + monitoramento = noites tranquilas. 🛡️
