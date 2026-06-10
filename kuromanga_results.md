# KUROMANGAS - Resultados do Ataque
## Site: kuromangas.com
- Cloudflare protegido (bloqueou todos os acessos)
- Backend via extensão Mihon/Tachiyomi identificado
- API endpoints mapeados: /api/mangas, /api/chapters, /api/auth/login
- CDN: cdn.kuromangas.com
- API: beta.kuromangas.com

## Tech Stack
- Frontend: WordPress + WP-Manga/Madara
- API: REST API (Node.js/Express)
- Auth: JWT Bearer Token
- CDN: Cloudflare

## Vulnerabilidades Encontradas (código-fonte)
- CRÍTICA: Debug Route /api/debug/debug/products sem auth
- CRÍTICA: Credenciais hardcoded: admin@mangastore.com / admin123
- ALTA: CORS mal configurado (permite Origin: null)
- ALTA: Verbose error leakage em development mode
- MÉDIA: Upload directory exposto
