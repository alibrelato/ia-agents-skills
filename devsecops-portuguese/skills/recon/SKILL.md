---
name: recon
description: Use no início de qualquer teste de segurança web, quando for preciso mapear a superfície de ataque antes de testar: subdomínios, DNS, portas e serviços, tecnologias, WAF, headers de segurança, endpoints, APIs, autenticação e roles. Fase de RECONHECIMENTO da metodologia do agente DevSecOps. Palavras-chave: recon, reconhecimento, enumeração, footprinting, subdomain, atack surface, OSINT, httpx, nmap, ffuf.
---

# Reconhecimento (Recon)

Mapear a superfície de ataque completa do alvo antes de qualquer teste de vulnerabilidade.

## Regras antes de começar
- Confirme que o alvo está dentro do escopo autorizado (contrato, ALP/RoL, bug bounty, lab).
- Não faça scans agressivos (velocidade alta, bypass de rate limit) exceto se autorizado.
- Registre a data/hora de cada atividade para compor as evidências do relatório.

## 1. Escopo e OSINT passivo
- Subdomínios: `subfinder -d target.com -all`, `amass enum -passive -d target.com`, consulta `crt.sh` (`https://crt.sh/?q=%25.target.com`).
- Resolução DNS: `dnsx -l subdomains.txt -a -cname -resp`, `dig`, `nslookup`.
- Google dorks / Shodan / Censys para assets expostos (somente leitura pública).
- Sempre salvar a lista de subdomínios em `recon/subdomains.txt`.

## 2. Recon ativo
- Probar os hosts vivos: `httpx -l subdomains.txt -title -tech-detect -status-code -server -location -o recon/live.txt`.
- Portas/serviços no alvo principal: `nmap -sV -sC --top-ports 1000 -oA recon/target` (e varredura completa em escopos pequenos).
- Fingerprint de tecnologia: `whatweb -a 3 URL`, extensão Wappalyzer, headers `Server`, `X-Powered-By`, cookies.
- Triple-handshake de tecnologias de deploy: contêiner/UUID, plataforma de CDN, framework.

## 3. Descoberta de conteúdo
- Diretórios/arquivos: `ffuf -w wordlist -u http://target.com/FUZZ -mc 200,204,301,302,307,401,403`.
- Wordlists úteis: `raft-large-directories.txt`, `directory-list-2.3-medium.txt`, `common.txt`.
- Arquivos especiais: `robots.txt`, `sitemap.xml`, `.well-known/*`, `favicon.ico` (hash), arquivos de backup (`.bak`, `.zip`, `.git/`), arquivos de documentação.
- Teste de exposição de `.git/` (git-dumper), `.env`, `.svn`, painéis administrativos.

## 4. Descoberta de API
- Buscar `/api`, `/api/v1`, `/swagger`, `/swagger-ui`, `/v3/api-docs`, `/openapi.json`, `/graphql`.
- Se GraphQL: testar `introspection` ativado.
- Coletar endpoints, parâmetros, rotas autenticadas/não autenticadas (método HTTP + roles).
- Registrar contratos de API em `recon/api-endpoints.md`.

## 5. WAF e infraestrutura
- Detectar WAF: `wafw00f URL`, analisar headers/cookies (`cf-ray`, `__cfduid`, `x-sucuri-id`, `set-cookie: xd_visitor`).
- Identificar CDN/cloud (Cloudflare, AWS CloudFront, Azure Front Door, Akamai, Incapsula).
- Mapear IP real por trás do CDN (history DNS, subdomínios sem proxy) somente se autorizado.
- Gravande neste skill `waf-analysis` para análise profunda e bypass.

## 6. Headers de segurança (baseline)
Auditar e documentar por host:
- `Strict-Transport-Security` (HSTS), `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, `X-XSS-Protection`.
- CORS: enviar request com `Origin: https://evil.com` e verificar espelhamento/acredita no `Access-Control-Allow-Origin`.

## 7. Autenticação e roles
- Localizar: páginas de login, fluxos SSO/OAuth/OIDC, JWTs (auditar algoritmo, exp, payload), cookies de sessão (flags `Secure`, `HttpOnly`, `SameSite`).
- Mapear roles/permissões visíveis (admin, user) e endpoints administrativos.
- Registrar em `recon/auth-surfaces.md` (login, register, reset password, MFA, API keys).

## 8. Organização das evidências
Estrutura obrigatória de saída (para alimentar as próximas skills e o relatório):
```
recon/
├── subdomains.txt
├── live.txt
├── nmap/            (target.nmap, target.gnmap, target.xml)
├── endpoints.md     (URLs, métodos, parâmetros, headers relevantes)
├── api-endpoints.md (contrato de API, autenticação necessária)
├── auth-surfaces.md (login/register/reset, tokens, roles)
└── notes.md         (tecnologias, WAF, CDN, CSP/CORS, achados preliminares)
```

## Saída esperada
Resumo compacto: lista de hosts vivos, tecnologias identificadas, WAF/CDN presente, endpoints descobertos, superfícies de autenticação, e a estrutura `recon/` populada como base para os testes das skills `waf-analysis`, `owasp-web-testing`, `llm-agentic-testing` e `exploitation`.