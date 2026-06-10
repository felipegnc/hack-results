# ZARION APPLICATIONS - Resultados do Ataque
## Acessos Conquistados
- Login via email: josephfelipegusmao09@gmail.com
- JWT: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9... (válido até 16/06)
- Discord: felipegnc13 (ID: 534773316938891273)
- Bot Discord: Big Account's (ID: 1450011521550913628)

## Infraestrutura
- Frontend: Next.js + Vercel + Cloudflare
- Backend: Express.js (cloud.zarionapplications.com.br)
- Database: MongoDB (Mongoose)
- Auth: JWT HS256 + Discord OAuth2
- Rate Limit: DESLIGADO

## Vulnerabilidades Encontradas
- CRÍTICA: Rate limiting desabilitado (/api/health confirma)
- CRÍTICA: Stats públicos (/api/info/stats vaza faturamento)
- ALTA: NoSQL Injection via $exists no MongoDB
- MÉDIA: Discord OAuth sem PKCE
- MÉDIA: JWT sem revogação

## Endpoints Mapeados
- GET /api/apps
- POST /api/invoices/create (usa ObjectId MongoDB)
- POST /api/app/create (403 - precisa de plano)
- GET /api/auth/session
- WebSocket cloud.zarionapplications.com.br (token interno)
- GET /api/health, /api/info/stats, /api/info/plans
