---
name: appsec-audit
description: |
  Executa uma auditoria profunda de segurança (AppSec) em projetos de software, gerando um relatório técnico priorizado com no mínimo 30 riscos ou pontos de validação. Use este skill SEMPRE que o usuário quiser:
  - Fazer auditoria de segurança, pentest defensivo ou hardening de um projeto;
  - Encontrar vulnerabilidades em auth, permissões, tokens, dependências, banco de dados, frontend ou infra;
  - Gerar um relatório de riscos com severidade, impacto real e plano de correção;
  - Revisar supply chain, lockfile, CORS, configurações de produção ou isolamento entre tenants;
  - Proteger usuários finais contra vazamento de dados, abuso de sessão ou prompt injection.
  Palavras-chave que ativam: auditoria de segurança, AppSec, pentest, hardening, vulnerabilidades, OWASP, CVE, supply chain, RLS, autenticação, tokens, XSS, CSRF, SSRF, prompt injection, relatório de segurança, swarm de auditoria.
---

# AppSec Audit Skill

Audita projetos de software com profundidade de especialista sênior em AppSec e pentest defensivo. Gera relatório priorizado com no mínimo 30 achados, top 10 correções críticas, plano de execução em fases e checklists de proteção.

> **Premissa obrigatória**: Este skill pressupõe que o usuário é o dono do projeto ou tem autorização explícita para avaliá-lo. Nunca executar ações destrutivas, atacar sistemas externos ou expor segredos encontrados.

---

## Regras de Conduta do Agente

Antes de qualquer análise:

- **Mascarar** tokens, senhas, cookies, URLs assinadas e credenciais como `***` em qualquer output
- **Não executar** ações destrutivas, não atacar sistemas externos
- **Não alterar** código sem confirmação explícita do usuário
- **Não reproduzir** valores reais de `.env`, secrets ou chaves privadas — reportar existência e localização apenas
- **Primeiro reportar**, depois propor correção — nunca editar silenciosamente

---

## Workflow de Execução

### Fase 1 — Mapeamento seguro do projeto

```bash
# Estrutura geral — filtrar arquivos sensíveis
find . -maxdepth 5 -type f \
  | grep -v node_modules | grep -v ".git/" \
  | grep -v ".env" | grep -v "*.key" | grep -v "*.pem" \
  | sort

# Dependências
cat package.json 2>/dev/null | grep -E '"dependencies|devDependencies"' -A 999 | head -80
cat requirements.txt 2>/dev/null | head -60
cat Cargo.toml 2>/dev/null | grep -A30 '\[dependencies\]'

# Configurações expostas (sem imprimir valores)
grep -r "API_KEY\|SECRET\|PASSWORD\|TOKEN\|PRIVATE" --include="*.ts" --include="*.js" \
  --include="*.py" -l 2>/dev/null | head -20

# Lockfiles presentes?
ls package-lock.json yarn.lock pnpm-lock.yaml bun.lockb 2>/dev/null
```

Leia os arquivos de referência de domínio em `references/` para guiar a análise de cada categoria.

### Fase 2 — Análise por domínio

Cubra os 20 domínios de segurança listados em `references/domains.md`. Para cada domínio, procure ativamente padrões de risco no código, configuração e infraestrutura.

Use os padrões de detecção em `references/detection-patterns.md` para buscas específicas.

### Fase 3 — Geração do relatório

Para cada achado, use o formato padrão definido em `references/report-format.md`.

**Meta mínima: 30 achados.** Prioridade de severidade: Crítico → Alto → Médio → Baixo → Informativo.

### Fase 4 — Seções finais obrigatórias

Após os achados individuais, gerar obrigatoriamente:

1. **Top 10 correções mais importantes** (ordenadas por impacto × esforço)
2. **Plano de execução em fases**: Agora (críticos), Próxima sprint (altos), Depois (médios/informativos)
3. **Checklist: proteção contra libs com malware** (supply chain)
4. **Checklist: proteção de usuários finais**
5. **Testes automatizados recomendados** (com exemplos de tipo: unit, integration, e2e, SAST)
6. **Mudanças de produto/UX recomendadas** (permissões, modo projeto não-confiável, audit log, mascaramento)

---

## Referências

- `references/domains.md` — Os 20 domínios de segurança com o que procurar em cada um
- `references/detection-patterns.md` — Padrões grep/regex para detecção rápida de riscos
- `references/report-format.md` — Template de achado + formato das seções finais
- `references/severity-guide.md` — Critérios de severidade CVSS simplificado para tomada de decisão de produto

Leia todos os arquivos de referência no início da análise para maximizar cobertura.
