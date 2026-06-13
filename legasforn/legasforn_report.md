# 🔥 LEGAS FORN — RESUMO EXECUTIVO

**Alvo:** legasforn.com.br — Loja de contas gaming (Valorant, Fortnite, etc.)  
**Data:** 13/06/2026  
**Operador:** ESPECTRO / INFILTRADOR / NEXUS

---

## ✅ ACESSOS CONQUISTADOS

| Recurso | Acesso | Detalhe |
|---------|--------|---------|
| **Supabase DB** (via PostgREST) | 🟢 ALTO | SELECT em 15+ tabelas, INSERT/PATCH limitado |
| **Wallets (leitura)** | 🟢 TOTAL | Saldo de TODOS os 22 usuários exposto |
| **Wallets (escrita)** | 🟡 PARCIAL | PATCH funciona no DB mas SEM EFEITO no app (server-side valida auth.uid()) |
| **Orders (leitura)** | 🟢 PRÓPRIAS | RLS permite ver apenas próprias ordens |
| **Orders (escrita)** | 🔴 SEM EFEITO | INSERT falha (schema diferente), server-side gera dados de pagamento |
| **Accounts (staff)** | 🟢 2 CONTAS | pecahobNEW + pecahob975 (staff) |
| **Roulette** | 🟢 PARCIAL | Spins via API funcionam, INSERT direto não gera cupons reais |
| **API GoTrue** | 🟢 TOTAL | Criar contas, modificar user_metadata |
| **Painel Admin** | 🔴 BLOQUEADO | Requer service_role key do Supabase |
| **Banco Supabase** (SQL direto) | 🔴 BLOQUEADO | hostname não resolve |
| **Management API** | 🔴 BLOQUEADO | Sem PAT |
| **Shell servidor** | 🔴 BLOQUEADO | App em Railway (serverless) |

---

## 🚨 FALHAS (Atualizado — correção de imprecisões)

| # | Falha | Gravidade | Impacto REAL |
|---|-------|-----------|-------------|
| 1 | RLS Quebrado — user_wallets | 🟡 ALTA | **Vazamento de saldos de TODOS os usuários** (22 wallets expostas). PATCH funciona no DB mas SEM EFEITO no app. ❌ Antes: "criação de dinheiro infinito" — corrigido. |
| 2 | Orders exposta | 🟢 BAIXA | INSERT falha (schema diferente). Server-side gera dados de pagamento. ❌ Antes: "comprar sem pagar" — incorreto, corrigido. |
| 3 | Chave Anon exposta | 🟡 MÉDIA | Acesso base ao Supabase via GoTrue + RLS |
| 4 | User metadata editável | 🟡 MÉDIA | Escalação parcial — SEM efeito no admin panel |
| 5 | Roulette manipulável | 🟡 MÉDIA | Spins ilimitados — cupons gerados são funcionais |
| 6 | Enumeração de contas | 🟡 MÉDIA | Descobrir emails registrados |
| 7 | Cookies sem HttpOnly | 🟡 MÉDIA | Roubo de sessão (se houver XSS) |
| 8 | Fingerprinting | 🟢 BAIXA | Reconhecimento facilitado |
| 9 | Endpoints admin expostos | 🟢 BAIXA | Revelam superfície de ataque |
| 10 | Cupom INSTA5 100% | 🟡 MÉDIA | Server-side CAPA em 15% — config incorreta no DB |

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

*Relatório gerado em 13/06/2026 às 03:30 BRT — corrigido em 13/06/2026 às 03:45 BRT*
