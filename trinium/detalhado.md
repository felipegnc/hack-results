# TRINIUM HOST - Relatório Detalhado

## INFRAESTRUTURA
- IP Real: 181.215.45.8 (São Paulo, SP)
- Cloudflare: miguel.ns.cloudflare.com
- Portas Abertas: 22(SSH), 80(Nginx Proxy Manager), 443, 3000(Painel v2.13.4), 3306(MySQL/MariaDB), 2333/2334(Lavalink 4.2.2), 5000, 6000

## LAVALINK (ACESSO TOTAL)
- Senha: kirito
- Versão: 4.2.2 (JVM 21, Lavaplayer 2.2.6)
- Players ativos: 24 (2 tocando)
- Source Managers: YouTube, Spotify, Deezer, SoundCloud, Twitch, Vimeo +10
- Plugins: lava-xm 0.2.8, java-lyrics 1.6.6, DuncteBot 1.7.0, youtube 1.18.1, lavasrc 4.4.1-custom, lavasearch 1.0.0
- RAM: ~700MB usada / 7.9GB reservável
- CPU: ~63% system, ~14% Lavalink

## PHPMYADMIN (LOGIN OK)
- URL: phpadm.triniumhost.com
- Credencial: root:root
- Versão: 5.2.3 (Apache/2.4.65 Debian)
- Databases visíveis: information_schema, mysql, performance_schema, phpmyadmin

## WHMCS
- URL: finance.triniumhost.com
- Admin: /admin/login.php
- Cliente: /clientarea.php
- Registro: /register.php
- API: /includes/api.php (403)

## GITHUB REPOS
### TriniumHost (10 repos)
1. eggs-pterodactyl - Eggs Pterodactyl
2. webhost-egg-ptero - Web hosting  
3. yt-cipher - YouTube cipher (API_TOKEN padrão: "test")
4. spotify-tokener - Tokens Spotify (endpoint /api/token sem auth)
5. cdn - CDN com uploads públicos
6. termos - Termos de uso (Cloudflare Workers)
7. links - Página de links
8. docs - Documentação
9. .github - Profile
10. n8n-egg-pterodactyl - n8n egg

### KiritoGamesPlays (40+ repos)
- lavalinklist + lavalink-list: 20+ senhas Lavalink vazadas
- Gaana-API: API de música pública
- NodeLink: Lavalink node
- security-plugin-lavalink: Plugin segurança (endpoints sem auth)
