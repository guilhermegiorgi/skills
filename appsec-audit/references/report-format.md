# Formato do Relatório — AppSec Audit

## Template de Achado Individual

```
---

**ID**: SEC-[NNN]
**Título**: [Nome curto descritivo]
**Severidade**: 🔴 Crítico | 🟠 Alto | 🟡 Médio | 🔵 Baixo | ℹ️ Informativo
**Domínio**: [ex: Autenticação, Supply Chain, SSRF...]
**Localização**: [arquivo:linha ou componente/módulo]

**Como isso poderia afetar o usuário:**
[Explicação em linguagem acessível — o que um usuário real perderia ou sofreria se isso fosse explorado. Ex: "Um atacante poderia acessar os dados de qualquer outro usuário do sistema apenas trocando um número na URL."]

**Como um atacante poderia abusar disso:**
[Cenário de ataque em alto nível, sem detalhes que permitam reprodução imediata. Ex: "Com acesso ao token de sessão (via log ou rede não-criptografada), o atacante pode se autenticar como qualquer usuário sem precisar da senha."]

**Como corrigir:**
[Passos concretos e priorizados. Incluir snippet de código ou configuração quando relevante, sem expor secrets reais.]

**Teste recomendado para provar que foi corrigido:**
[Comando, script, request HTTP ou procedimento de verificação. Ex: "Tentar acessar /api/users/[id-de-outro-usuario] com token do usuário A — deve retornar 403."]
```

---

## Seções Finais Obrigatórias

### 1. Top 10 Correções Mais Importantes

Tabela ordenada por **impacto × facilidade de exploração**:

| # | ID | Título | Severidade | Esforço estimado |
|---|-----|--------|-----------|-----------------|
| 1 | SEC-XXX | ... | 🔴 Crítico | Horas |
| ... | | | | |

### 2. Plano de Execução em Fases

**🚨 Agora (esta semana — Críticos e Altos bloqueadores)**
- SEC-XXX: [título] — [ação específica]
- ...

**📅 Próxima Sprint (Altos não-bloqueadores e Médios de alto impacto)**
- SEC-XXX: [título] — [ação específica]
- ...

**🗓️ Depois (Médios restantes, Baixos, melhorias)**
- SEC-XXX: [título] — [ação específica]
- ...

---

### 3. Checklist: Proteção contra Libs com Malware

```
[ ] npm audit / pip-audit / cargo audit em CI — falha no build se crítico
[ ] Lockfile commitado e verificado (--frozen-lockfile em CI)
[ ] Dependências de produção separadas das de desenvolvimento
[ ] Revisar post-install scripts antes de instalar novo pacote
[ ] Verificar nome de pacotes novos contra typosquatting
[ ] Monitoramento contínuo de CVEs (Dependabot, Snyk, Socket.dev)
[ ] Revisar permissões de pacotes (acesso a FS, rede, env) antes de adicionar
[ ] Não usar * ou latest como versão em dependências de produção
[ ] GitHub Actions com hash de commit fixo (não @v3 dinâmico)
[ ] SBOM (Software Bill of Materials) gerado a cada release
```

---

### 4. Checklist: Proteção de Usuários Finais

```
[ ] HTTPS forçado em todas as rotas (HSTS habilitado)
[ ] Cookies com Secure + HttpOnly + SameSite=Strict/Lax
[ ] Rate limiting em login, signup, reset de senha e APIs críticas
[ ] Mensagens de erro genéricas (sem revelar existência de usuário)
[ ] Logout invalida sessão server-side
[ ] Notificação ao usuário em login de novo dispositivo/IP
[ ] MFA disponível para contas com dados sensíveis
[ ] Dados pessoais criptografados em repouso
[ ] Direito de exclusão de conta implementado (LGPD/GDPR)
[ ] Audit log de ações sensíveis acessível ao usuário (quando aplicável)
[ ] Consentimento registrado antes de coleta de dados
[ ] PII mascarada em logs e error tracking
```

---

### 5. Testes Automatizados Recomendados

**SAST (Static Analysis)**
- Integrar: `semgrep`, `eslint-plugin-security`, `bandit` (Python), `gosec` (Go)
- Rodar em CI em todo PR

**Dependency Scanning**
- `npm audit --audit-level=high` no CI (falha se alto/crítico)
- `dependabot` ou `renovate` para PRs automáticos de updates

**Autenticação / Autorização (integration tests)**
```
- Testar acesso a recurso de outro usuário → espera 403
- Testar endpoint admin sem role → espera 403
- Testar token expirado → espera 401
- Testar CSRF sem token → espera 403
- Testar rate limit em /login → espera 429 após N tentativas
```

**Input Validation (unit tests)**
```
- Path traversal: input "../../../etc/passwd" → deve rejeitar
- SQL injection: input "' OR '1'='1" → deve escapar/rejeitar
- XSS: input "<script>alert(1)</script>" → deve sanitizar no output
```

**Headers de Segurança (e2e / smoke tests)**
```
- Verificar presença de: Content-Security-Policy, X-Frame-Options,
  X-Content-Type-Options, Strict-Transport-Security, Permissions-Policy
- Verificar ausência de: Server version, X-Powered-By
```

---

### 6. Mudanças de Produto/UX Recomendadas

**Permissões e acesso**
- Implementar modelo de permissões explícitas (não implícitas por cargo)
- Tela de "Revisar permissões do workspace" visível ao usuário
- Acesso de terceiros/integrações com escopos mínimos declarados

**Modo projeto não-confiável**
- Alertar o usuário ao abrir/executar projeto de fonte desconhecida
- Sandboxing de execução de código não-confiável (sem acesso a FS/rede por padrão)
- Confirmação explícita antes de executar scripts de instalação

**Confirmação de ações sensíveis**
- Confirmação com senha/MFA antes de: deletar conta, exportar dados, transferir ownership
- Janela de desfazer para ações destrutivas (grace period)

**Audit log para o usuário**
- Página "Atividade da conta" com: logins, dispositivos, ações críticas
- Notificação por email em eventos críticos (novo login, mudança de senha, export)

**Mascaramento de segredos**
- Na UI, mascarar tokens e API keys por padrão (mostrar só ao criar)
- Detectar e alertar se usuário colar token em campo de texto livre
- Revogar automaticamente tokens detectados em commits públicos (via integração GitHub/GitLab)
