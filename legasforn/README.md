# 🎯 Legas Forn - Pentest Report

**Alvo:** legasforn.com.br (69.46.46.108)  
**Infra:** Railway (Next.js 14 Turbopack → Supabase + BassPag PIX + Vercel Blob)  
**Data:** 13/06/2026  
**Status:** 🟡 Acesso Parcial (painel admin bloqueado — sem service_role key)

---

## 📋 Sumário das Falhas

| # | Falha | Gravidade | CVE-like | Status |
|---|-------|-----------|----------|--------|
| 1 | **RLS Quebrado — user_wallets sem auth.uid()** | 🔴 CRÍTICA | OWASP API9 | ✅ Explorado |
| 2 | **Criação de Orders sem Pagamento** | 🔴 CRÍTICA | OWASP API4 | ✅ Explorado |
| 3 | **Chave Anon do Supabase Exposta** | 🟡 Média | - | ✅ Confirmado |
| 4 | **Metadados de Usuário Modificáveis via GoTrue** | 🟡 Média | OWASP API2 | ✅ Explorado |
| 5 | **Roda da Sorte Manipulável via API** | 🟡 Média | OWASP API3 | ✅ Confirmado |
| 6 | **Enumeração de Usuários via Signup** | 🟡 Média | OWASP API1 | ✅ Confirmado |
| 7 | **Cookies de Autenticação sem HttpOnly** | 🟡 Média | - | ✅ Confirmado |
| 8 | **Tech Stack & Build ID Expostos (fingerprinting)** | 🟢 Baixa | - | ✅ Confirmado |
| 9 | **Endpoints Admin Expostos (403/401)** | 🟢 Baixa | - | ✅ Confirmado |
| 10 | **Cupom de 100% sem limite de uso** | 🟡 Média | - | Confirmado |
