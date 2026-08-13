---
name: waf-analysis
description: Use quando o alvo tiver WAF ou ModSecurity e for preciso identificar as regras ativas, falsos positivos, técnicas de bypass e logs de detecção. Cubre Cloudflare WAF, AWS WAF, Azure WAF, Nginx + ModSecurity (OWASP CRS). Palavras-chave: WAF, ModSecurity, CRS, bypass, evasão, Cloudflare, AWS WAF, Azure WAF, wafw00f, regras de detecção, paranoia level.
---

# Análise de WAF / ModSecurity

Identificar o WAF, mapear as regras ativas e avaliar técnicas de evasão de forma controlada.

## 1. Identificação do WAF
- `wafw00f URL` para fingerprint.
- Indicadores em resposta:
  - Headers: `cf-ray`, `__cfduid` (Cloudflare); `x-sucuri-id`, `x-cdn` (Sucuri); `x-azure-ref` (Azure Front Door); `x-aws-*` (CloudFront/AWS); `server: cloudflare`.
  - Resposta de bloqueio: página com ID de incidente/`cf-error`, status 403/406/418/429, challenge JS/captcha.
  - Cookies: `__cf_bm`, `AWSALB`, `AzureAdSessionCookie`, `_cflb`.
- Registrar fabricante/versão e se está em modo detect ou block.

## 2. Enumerar regras ativas
- Para ModSecurity/CRS: identificar as paranoia levels ativas (PL1–PL4) testando payloads básicos que disparam regras de cada nível.
- Testar categorias de regras: SQLi (930xxx), XSS (941xxx), LFI/Path Traversal (930/931), Command Injection (932xxx), Java/deserialização (934xxx), RCE genérico (932xxx), protocolo (920xxx).
- Diferenciar bloqueio por regra de detecção vs. rate limit vs. challenge (comportamento de resposta).

## 3. Técnicas de bypass (testar cada uma com request válido e inválido)
Todas as técnicas abaixo devem ser testadas em payloads reais e documentadas (blocked = resposta do WAF, passed = resposta da aplicação):

- **Encoding**: URL double encoding (`%2527`), full-width/Unicode (`％２７`), encoding HTML (`&#x27;`), base64/hex em parâmetros interpretados.
- **Case variation**: `SeLeCt`, `UNION sElEcT` quando o backend é case-insensitive.
- **Comment injection**: `UN/**/ION SE/**/LECT`, `a/**/b`, `%0a`/`%0d` (newline/tab `%09`) dentro de payloads.
- **Null byte/truncation**: `%00` em argumentos (raros em HTTP, comum em filesystem).
- **HTTP Parameter Pollution (HPP)**: repetir parâmetros (`?id=1&id=1 OR 1=1`), argumento em JSON+form, `id[]`, parâmetros com nomes duplicados (WAF analisa só o primeiro, backend usa o último).
- **HTTP method override**: `X-HTTP-Method-Override`, `X-Method-Override`, `_method`, método `POST` para payloads que o WAF só inspeciona em `GET` (e vice-versa).
- **Chunked Transfer-Encoding**: enviar corpo em chunks (`Transfer-Encoding: chunked`), técnica clássica contra ModSecurity em servidores antigos.
- **Content-Type confusion**: enviar payload com `Content-Type: multipart/form-data`, `application/json`, `text/plain`, `application/x-www-form-urlencoded` e observar qual o WAF inspeciona.
- **Protocolo/fronteira**: downgrade HTTP/2 → HTTP/1.1, reutilização de conexão, variações de `Host`/`User-Agent`, `path` com `/./` e `//`, trailer headers.
- **Oversized/limites**: enviar payloads acima do `SecRequestBodyLimit` para causar bypass por "truncation" do WAF (só se o backend aceitar corpo maior).
- **WebSocket / API interna**: endpoints que o WAF não inspeciona (WebSocket, endereços internos, `/admin`, IP direto).

## 4. Validação de bypass
- Confirme que o payload **de fato** chegou à aplicação: resposta com erro de SQL/sintaxe, mudança de comportamento, ou sucesso da operação.
- Um status 200 genérico NÃO é prova de bypass — valide a resposta.
- Cruze cada bypass com os logs de detecção (se acessíveis) para confirmar que a regra não disparou.

## 5. Falsos positivos
- Diferenciar: bloqueio por WAF vs. erro de aplicação (404 genérico, 500 sem corpo de detecção) vs. rate limit (429/retry-After) vs. challenge (JS/captcha).
- Registrar quais regras geram falso positivo em tráfego legítimo.

## 6. Logs de detecção
- Onde procurar: ModSecurity (`SecAuditLog`), logs de acesso do Nginx, dashboard/analytics do WAF (Cloudflare analytics, AWS WAF sampled requests), alertas SIEM.
- Padrões de log a registrar: rule ID, `SecRule ID:932230`, URI, origem, timestamp, payload ofensivo detectado.

## 7. Documentação
Saída obrigatória:
```
waf/
├── fingerprint.md      (fabricante, versão, modo, IP real se mapeado)
├── rules-detected.md   (IDs de regras/paranoia levels identificados)
├── bypass-validated.md (cada técnica: payload, request completo, blocked/passed, evidência)
└── fp-analysis.md      (falsos positivos e comportamento de bloqueio)
```
Cada bypass validado deve incluir request HTTP completo (headers + body) e trecho da resposta, para reutilização nas skills `owasp-web-testing` e `exploitation`.