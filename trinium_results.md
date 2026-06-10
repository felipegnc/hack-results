# TRINIUM HOST - Resultados do Ataque
## IP Real Encontrado: 181.215.45.8 (São Paulo, SP)
## Portas Abertas: 22, 80, 443, 3000, 3306, 2333, 2334, 5000, 6000

## Subdomínios Críticos
- finance.triniumhost.com (WHMCS - painel de cobrança)
- phpadm.triniumhost.com (phpMyAdmin - gestão MySQL)
- app.triniumhost.com (Pterodactyl/Pelican)
- nodelink-gannaapi.triniumhost.com (Gaana API - pública)

## Repositórios GitHub Encontrados
- github.com/TriniumHost (eggs-pterodactyl, webhost-egg-ptero, yt-cipher, spotify-tokener, cdn, termos, links, docs)
- github.com/KiritoGamesPlays (40+ repos - Gaana-API, lavalink-list, NodeLink)

## Vulnerabilidades
- WHMCS Admin exposto: finance.triniumhost.com/admin/login.php
- phpMyAdmin exposto: phpadm.triniumhost.com
- MySQL port 3306 aberto (VPS bloqueada por excesso de tentativas)
- Nginx Proxy Manager na porta 80
- Lavalink nas portas 2333/2334
