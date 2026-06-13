# 🎯 Legas Forn - Pentest Report

**Alvo:** legasforn.com.br (69.46.46.108)  
**Infra:** Railway (Next.js 14 Turbopack → Supabase + BassPag PIX + Vercel Blob)  
**Data:** 13/06/2026  
**Status:** 🟡 Acesso Parcial (painel admin bloqueado — sem service_role key)

---

## 📋 Sumário das Falhas

| # | Falha | Gravidade | Impacto REAL | 
|---|-------|-----------|-------------|
| 1 | **RLS Quebrado — user_wallets sem auth.uid()** | 🟡 ALTA | Vazamento de saldo de TODOS os 22 usuários + PATCH sem efeito no app |
| 2 | **Tabela orders exposta** | 🟢 BAIXA | INSERT falha (schema real é complexo, não há coluna "items") |
| 3 | **Chave Anon do Supabase Exposta** | 🟡 Média | Acesso base via GoTrue |
| 4 | **Metadados de Usuário Modificáveis via GoTrue** | 🟡 Média | Escalação parcial (sem efeito no admin panel) |
| 5 | **Roda da Sorte Manipulável via API** | 🟡 Média | Spins ilimitados, cupons funcionais gerados |
| 6 | **Enumeração de Usuários via Signup** | 🟡 Média | Descobrir emails registrados |
| 7 | **Cookies de Autenticação (confirmar HttpOnly)** | 🟡 Média | Potencial roubo de sessão |
| 8 | **Tech Stack & Build ID Expostos (fingerprinting)** | 🟢 Baixa | Reconhecimento facilitado |
| 9 | **Endpoints Admin Expostos (403/401)** | 🟢 Baixa | Revelam superfície de ataque |
| 10 | **Cupom INSTA5 100% (server-side capado em 15%)** | 🟡 Média | Config incorreta no DB, mas server-side protege |

---

## ⚠️ Nota sobre Correções

As Falhas #1 e #2 foram **corrigidas neste relatório** em relação à versão anterior. A classificação original como "CRÍTICA — criação de dinheiro infinito" era imprecisa:

- **Falha #1**: O RLS está quebrado (vazamento de dados REAL), mas PATCH direto no DB **NÃO** altera o saldo exibido pelo app. O server-side (`/api/wallet`) valida `auth.uid()` corretamente.
- **Falha #2**: A tabela `orders` não tem coluna `items`, e INSERT direto falha. Server-side gera dados de pagamento complexos.
