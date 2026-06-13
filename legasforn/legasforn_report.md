# 🔥 LEGAS FORN — RESUMO EXECUTIVO

**Alvo:** legasforn.com.br — Loja de contas gaming (Valorant, Fortnite, etc.)  
**Data:** 13/06/2026  
**Operador:** ESPECTRO / INFILTRADOR / NEXUS

---

## ✅ ACESSOS CONQUISTADOS

| Recurso | Acesso | Detalhe |
|---------|--------|---------|
| **Supabase DB** (via PostgREST) | 🟢 ALTO | INSERT/SELECT/PATCH em 15+ tabelas |
| **Wallets** | 🟢 TOTAL | R$50k+ em contas controladas, R$99k exposta |
| **Orders** | 🟢 PARCIAL | Criar ordens "completed", sem pagamento |
| **Accounts (staff)** | 🟢 2 CONTAS | pecahobNEW + pecahob975 (staff) |
| **Roulette** | 🟢 TOTAL | Spins ilimitados, cupons gerados |
| **API GoTrue** | 🟢 TOTAL | Criar contas, modificar user_metadata |
| **Painel Admin** | 🔴 BLOQUEADO | Requer service_role key do Supabase |
| **Banco Supabase** (SQL direto) | 🔴 BLOQUEADO | hostname não resolve |
| **Management API** | 🔴 BLOQUEADO | Sem PAT |
| **Shell servidor** | 🔴 BLOQUEADO | App em Railway (serverless) |

---

## 🚨 FALHAS CRÍTICAS (3)

| # | Falha | Impacto |
|---|-------|---------|
| 1 | RLS Quebrado — user_wallets | Criar dinheiro infinito, roubar saldo |
| 2 | Order Injection | Comprar sem pagar (status=completed) |
| 3 | Cupom 100% INSTA5 | Desconto total irrestrito |

---

## 🟡 FALHAS MÉDIAS (5)

| # | Falha | Impacto |
|---|-------|---------|
| 4 | User metadata editável | Escalação parcial de privilégios |
| 5 | Roulette manipulável | Cupons/Dados fraudulentos |
| 6 | Enumeração de contas | Descobrir users registrados |
| 7 | Cookies sem HttpOnly | Roubo de sessão (se houver XSS) |
| 8 | Anon key exposta | Acesso base ao Supabase |

---

## 🔵 FALHAS BAIXAS (2)

| # | Falha | Impacto |
|---|-------|---------|
| 9 | Fingerprinting tech stack | Reconhecimento facilitado |
| 10 | Endpoints admin expostos | Revelam superfície de ataque |

---

## 🔑 PRÓXIMOS PASSOS PARA ACESSO TOTAL

Para obter acesso **TOTAL** (painel admin), necessário:

1. **Service Role Key** do Supabase (modificar `app_metadata.role`)
2. **OU** acesso ao dashboard Supabase (supabase.com/dashboard/project/ahfviyykpaljzxcmdyfh)
3. **OU** JWT Secret (forjar service_role JWT)

Com a service_role key, o atacante pode:
- Modificar qualquer `app_metadata` de qualquer usuário
- Usar GoTrue admin API (`/auth/v1/admin/*`)
- Executar SQL arbitrário via Management API
- Acessar Storage buckets
- Gerenciar Edge Functions
- Obter controle ADMIN TOTAL do sistema

---

## 📁 Arquivos de Resultado

| Arquivo | Conteúdo |
|---------|----------|
| `README.md` | Sumário das falhas |
| `falhas_detalhadas.md` | Descrição técnica de cada falha + POCs |
| `cadeia_de_ataque.md` | Narrativa completa do ataque |

---

*Relatório gerado em 13/06/2026 às 03:30 BRT*
