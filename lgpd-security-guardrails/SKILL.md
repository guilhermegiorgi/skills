---
name: lgpd-security-guardrails
description: |
  Cria ou atualiza uma camada permanente de guardrails de segurança, LGPD/privacidade e responsabilidade em projetos de software. Use este skill SEMPRE que o usuário quiser:
  - Adicionar proteções de segurança, LGPD ou privacidade a um projeto;
  - Criar/atualizar AGENTS.md, CLAUDE.md, docs de compliance ou guardrails para agentes de IA;
  - Auditar um projeto em busca de gaps de segurança, RLS, dados pessoais, aceite de termos ou segredos expostos;
  - Garantir que futuros agentes de IA não quebrem auth, checkout, termos, consentimento ou banco de dados.
  Palavras-chave que ativam: guardrails, LGPD, segurança, privacidade, compliance, AGENTS.md, CLAUDE.md, RLS, dados pessoais, aceite, termos, secrets, auditoria de segurança, proteção de dados.
---

# LGPD & Security Guardrails Skill

Cria proteção permanente em projetos — um arquivo `docs/agent-legal-privacy-guardrails.md` que futuros agentes de IA obrigatoriamente devem consultar antes de tocar em áreas sensíveis.

---

## Workflow de Execução

### 1. Leitura segura do projeto

Antes de qualquer análise, aplique as regras de leitura segura:

```bash
# Leia estrutura geral — NÃO abra .env nem arquivos de secrets
find . -maxdepth 4 -type f \
  | grep -v node_modules \
  | grep -v ".git/" \
  | grep -v ".env" \
  | grep -v "secrets" \
  | grep -v "*.key" \
  | grep -v "*.pem" \
  | sort
```

Procure especificamente por:
- `AGENTS.md`, `CLAUDE.md`, `README.md`
- `docs/`, `migrations/`, `supabase/`
- Pastas: `auth/`, `checkout/`, `payments/`, `privacy/`, `terms/`, `community/`, `support/`
- Arquivos: `*.sql`, `schema.*`, `prisma/schema.prisma`

**Nunca abra nem imprima:** `.env`, `.env.*`, arquivos com `secret`, `token`, `credential`, `cookie`, `private_key` no nome. Se um comando puder expor segredo, filtre antes de imprimir.

### 2. Análise dos gaps

Documente o que encontrou em cada categoria:

| Categoria | Encontrado | Gap identificado |
|-----------|-----------|-----------------|
| AGENTS.md / CLAUDE.md | sim/não | tem/não tem regra de guardrails |
| Migrations/RLS | sim/não | tabelas sem RLS habilitado |
| Auth / Checkout | sim/não | presença de checkbox aceite |
| Termos / Política | sim/não | links preservados |
| Logs sensíveis | sim/não | body/headers completos logados |
| Secrets hardcoded | sim/não | tokens/keys no código |

### 3. Criação do arquivo principal

Crie `docs/agent-legal-privacy-guardrails.md` usando o template em `references/guardrails-template.md`.

Personalize com base no stack encontrado (ex: se usa Supabase, enfatizar RLS; se tem checkout, enfatizar aceite; se tem Discord/WhatsApp, enfatizar comunidade).

### 4. Atualização de AGENTS.md / CLAUDE.md

Se existir `AGENTS.md` ou `CLAUDE.md`, adicione no topo ou na seção de regras gerais:

```markdown
## Regra de Segurança e Privacidade

> **OBRIGATÓRIO**: Antes de alterar auth, cadastro, checkout, pagamento, comunidade,
> suporte, termos, privacidade, consentimento, logs, migrations ou coleta de dados
> pessoais, leia `docs/agent-legal-privacy-guardrails.md`.
```

Se não existir, crie `AGENTS.md` com essa regra como conteúdo inicial.

### 5. Entrega

Retorne:
- ✅ Arquivos criados/alterados (com diff resumido)
- 🛡️ O que foi protegido em cada categoria
- ⚠️ Gaps encontrados que precisam de ação humana
- 📋 Próximos passos recomendados
- 🔒 Confirmação de que nenhum secret/token/.env foi exposto

---

## Regras de Segurança para o Agente Durante a Execução

O agente executando este skill deve respeitar as mesmas regras que vai documentar:

1. **Nunca imprimir** conteúdo de `.env`, headers `Authorization`, cookies, tokens, API keys ou private keys
2. **Nunca gerar exemplos** com credenciais reais — usar sempre `YOUR_API_KEY`, `***`, ou `$ENV_VAR`
3. **Filtrar saída** de qualquer comando que possa expor secrets antes de mostrar ao usuário
4. **Não logar** body completo de requisições que contenham dados pessoais ou auth state
5. Se encontrar secrets hardcoded no código, **reportar a existência sem reproduzir o valor**

---

## Referências

- `references/guardrails-template.md` — Template completo do arquivo `docs/agent-legal-privacy-guardrails.md`
- `references/rls-checklist.md` — Checklist de RLS para Supabase/Postgres

Leia os arquivos de referência ao criar os arquivos no projeto do usuário.
