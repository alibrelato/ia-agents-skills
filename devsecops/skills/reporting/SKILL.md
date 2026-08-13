---
name: reporting
description: Use no final do trabalho, quando todos os testes e explorações estiverem concluídos, para gerar o relatório técnico no formato padronizado do agente DevSecOps: título, severidade com CVSS, ativo afetado, descrição técnica, evidências, PoC passo a passo, impacto, recomendações de correção e referências. Gera também o resumo executivo e os anexos de payloads e logs. Palavras-chave: relatório, report, documento, markdown, CVSS, resumo executivo, PoC, severidade, severidade crítica.
---

# Relatório Técnico

Consolidar todos os achados no relatório padronizado do agente, reprodutível e acionável. Esta skill consome os artefatos das skills `recon`, `waf-analysis`, `owasp-web-testing`, `llm-agentic-testing` e `exploitation`.

## Estrutura obrigatória do relatório

Para CADA vulnerabilidade encontrada, gerar as seguintes seções:

### 1. Título
- Formato: `<Classe de vulnerabilidade> <contexto> em <alvo>` (ex.: "SQL Injection autenticada em /api/v1/users (POST)").

### 2. Severidade
- Nível: Critical / High / Medium / Low / Informational.
- CVSS 3.1/4.0 quando aplicável: vetor completo + score (ex.: `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` → 7.5).
  - Calcular com o FIRST CVSS Calculator ou software oficial; justificar cada métrica.

### 3. Ativo afetado
- URL/IP/hostname, aplicação, componente, ambiente (prod/staging/lab) e versão quando identificada.

### 4. Descrição técnica
- O que é, por que ocorre (causa raiz), contexto (autenticação, role, parâmetros, método HTTP, headers relevantes).

### 5. Evidências
- Request HTTP completo (headers + body), trecho da response relevante, screenshots (se houver), hashes SHA256 de payloads críticos, timestamps.

### 6. Prova de conceito (PoC) — Reprodução
- Passos numerados e copiáveis (incluindo credenciais de teste e token de exemplo), re-executáveis em ambiente idêntico:
  ```bash
  curl -X POST 'https://target.com/api/v1/users' \
    -H 'X-Auth-Token: <token>' \
    -H 'Content-Type: application/json' \
    -d '{"id": "1 OR 1=1--"}'
  ```
- Indicar o resultado esperado que comprova a vulnerabilidade.

### 7. Impacto
- O que um atacante alcança (ex.: acesso total ao DB, RCE, roubo de sessão, PII exposta).
- Impacto de negócio: financeiro, reputacional, compliance (LGPD/GDPR, PCI-DSS, etc.).

### 8. Recomendações de correção
- Correção específica e exemplos de código (prepared statements, validação de input, CSP, correção de CORS, etc.).
- Priorização por severidade (fix crítico primeiro) e referências (OWASP, CWE, vendor advisories).

### 9. Referências
- OWASP Top 10 (item e link), CWE ID, CVEs relacionados, documentação do fornecedor.

## Resumo executivo (1 página)
Para stakeholders não técnicos:
- Contexto do teste (escopo, autorização, período).
- Número de achados por severidade (tabela).
- Top 3 riscos em linguagem simples.
- Recomendações prioritárias.
- Status de risco geral (ex.: Alto).

## Anexos obrigatórios
1. **Lista de payloads testados** (`payloads.csv` ou `.json`): campo do payload, técnica, status (blocked/passed/validated), hash SHA256, referência ao achado.
2. **Logs de ferramentas**: outputs de nuclei, sqlmap, nmap, ffuf, wafw00f etc. (resumidos, sem dados sensíveis).
3. **Glossário** de siglas (se houver público misto).

## Redação e integridade
- Mascarar qualquer dado sensível real (senhas, tokens, PII) com `***`.
- Não incluir instruções que permitam explorar sistemas sem autorização.
- Ordenar achados por severidade (Critical → Informational).
- Nomear arquivo padrão: `report/<alvo>-pentest-report.md` (ou `.pdf` via conversão), acompanhado de `executive-summary.md` e os anexos.

## Verificação final (checklist)
- [ ] Toda vulnerabilidade tem as 9 seções.
- [ ] Cada PoC foi testado/validado (não teórico).
- [ ] Severidade + CVSS calculados.
- [ ] Evidências sem dados sensíveis reais.
- [ ] Resumo executivo e anexos gerados.
- [ ] Metodologia/ferramentas/versões documentadas (PTES, OWASP Testing Guide, NIST SP 800-115, OSSTMM).