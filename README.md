# gg.ai skills

[![skills.sh](https://skills.sh/b/guilhermegiorgi/skills)](https://skills.sh/guilhermegiorgi/skills)

Skills para agentes de IA focados em segurança, compliance e boas práticas de engenharia de software. Compatíveis com Claude Code, Cursor, Codex, Windsurf e [outros agentes](https://github.com/vercel-labs/skills#supported-agents).

---

## Instalação

```bash
# Instalar todas as skills
npx skills add guilhermegiorgi/skills

# Instalar uma skill específica
npx skills add guilhermegiorgi/skills --skill appsec-audit
npx skills add guilhermegiorgi/skills --skill lgpd-security-guardrails

# Instalar globalmente (disponível em todos os projetos)
npx skills add guilhermegiorgi/skills -g

# Instalar para um agente específico
npx skills add guilhermegiorgi/skills -a claude-code
```

---

## Skills disponíveis

### 🔒 `appsec-audit`

Auditoria profunda de segurança com visão de especialista sênior em AppSec e pentest defensivo. Gera um relatório técnico e priorizado cobrindo 20 domínios de segurança.

**Quando usar:** auditoria de segurança, pentest defensivo, hardening de projeto, revisão de vulnerabilidades antes de release, due diligence de segurança.

**O que entrega:**
- Relatório com no mínimo 30 achados no formato: ID, título, impacto real para o usuário, vetor de abuso, como corrigir e teste de validação
- Top 10 correções críticas priorizadas por impacto × esforço
- Plano de execução em 3 fases: agora, próxima sprint, depois
- Checklist de proteção contra libs maliciosas (supply chain)
- Checklist de proteção de usuários finais
- Testes automatizados recomendados (SAST, dependency scanning, integration tests)
- Mudanças de produto/UX recomendadas (audit log, modo projeto não-confiável, mascaramento de secrets)

**Domínios cobertos:** autenticação e sessões · autorização e multi-tenant · exposição de tokens · dependências vulneráveis · scripts de instalação · shell injection · path traversal · SSRF · upload/download · logs e vazamento · SQL injection · XSS/CSRF · prompt injection · supply chain · CORS e configurações de produção · isolamento de ambientes · auditoria e rastreabilidade

```bash
npx skills add guilhermegiorgi/skills --skill appsec-audit
```

---

### 🛡️ `lgpd-security-guardrails`

Cria uma camada permanente de proteção em projetos — um arquivo `docs/agent-legal-privacy-guardrails.md` que futuros agentes de IA devem consultar antes de tocar em áreas sensíveis.

**Quando usar:** adicionar proteções de LGPD/privacidade a um projeto, criar/atualizar `AGENTS.md` ou `CLAUDE.md` com regras de segurança, auditar RLS em Supabase/Postgres, garantir que agentes não quebrem auth, checkout ou consentimento.

**O que entrega:**
- `docs/agent-legal-privacy-guardrails.md` com regras obrigatórias para agentes futuros
- Atualização de `AGENTS.md` / `CLAUDE.md` com regra de consulta obrigatória
- Checklist de RLS com queries SQL prontas para Supabase/Postgres
- Template de registro de aceite de termos com hash de IP
- Mapeamento de gaps por categoria com resumo de próximos passos

**Categorias cobertas:** secrets e hardcoding · dados pessoais e LGPD · aceite e termos · IP/user-agent e prova de aceite · RLS e banco de dados · comunidade e suporte

```bash
npx skills add guilhermegiorgi/skills --skill lgpd-security-guardrails
```

---

## Uso

Após instalar, o agente detecta automaticamente quando aplicar cada skill. Você também pode invocar diretamente:

```
# No chat do agente:
"Faz uma auditoria de segurança completa nesse projeto"
"Adiciona guardrails de LGPD e segurança ao projeto"
"Revisa o projeto em busca de vulnerabilidades antes do deploy"
```

---

## Estrutura do repositório

```
skills/
├── appsec-audit/
│   ├── SKILL.md
│   └── references/
│       ├── domains.md              # 20 domínios de segurança detalhados
│       ├── detection-patterns.md  # Comandos grep/bash para detecção rápida
│       ├── report-format.md       # Template de achado + seções finais
│       └── severity-guide.md      # Critérios Crítico/Alto/Médio/Baixo
└── lgpd-security-guardrails/
    ├── SKILL.md
    └── references/
        ├── guardrails-template.md  # Template do docs/agent-legal-privacy-guardrails.md
        └── rls-checklist.md        # Checklist RLS com queries SQL
```

---

## Sobre

Desenvolvido por [Guilherme Giorgi](https://github.com/guilhermegiorgi) — Engenheiro Agrônomo, especialista em IA aplicada e desenvolvimento de sistemas.  
[gg.ai labs](https://ggailabs.com) · Nova Mutum, MT.
