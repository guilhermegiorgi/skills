# Guia de Severidade — AppSec Audit

Critérios simplificados para tomada de decisão de produto e engenharia.
Baseado em CVSS v3, adaptado para contexto prático.

---

## 🔴 Crítico

**Quando usar**: Exploração remota sem autenticação, acesso a dados de todos os usuários,
execução de código arbitrário, bypass completo de autenticação.

**Impacto de produto**: Vazamento massivo de dados, compliance breach (LGPD/GDPR),
dano à reputação irreversível, risco legal imediato.

**Exemplos**:
- SQL injection que expõe toda a base de usuários
- JWT `alg: none` aceito (qualquer um forja token)
- Chave privada exposta publicamente
- RCE via input não sanitizado
- IDOR que permite acessar dados de qualquer usuário

**Ação**: Corrigir imediatamente. Considerar hotfix em produção. Notificar usuários afetados se necessário.

---

## 🟠 Alto

**Quando usar**: Exploração requer autenticação ou condição específica, mas impacto
é severo — acesso a dados de outros usuários, escalada de privilégio, DoS.

**Impacto de produto**: Usuários específicos afetados, risco de perda de dados,
potencial exposição de PII, possível violação de compliance.

**Exemplos**:
- IDOR limitado a usuários do mesmo tenant
- XSS armazenado em área crítica (ex: comentários de admin)
- CSRF em operações financeiras ou de configuração
- Dependência com CVE high/critical em uso ativo
- Secrets em logs acessíveis por outros membros do time

**Ação**: Corrigir na sprint atual ou próxima. Não bloquear release, mas incluir no roadmap imediato.

---

## 🟡 Médio

**Quando usar**: Exploração requer múltiplas condições ou impacto é limitado em escopo.
Pode ser amplificado por outros problemas.

**Impacto de produto**: Usuário individual afetado, informações internas expostas,
degradação de experiência de segurança.

**Exemplos**:
- Enumeração de usuários via mensagem de erro
- Rate limiting ausente em endpoints não-críticos
- Headers de segurança faltando (CSP, HSTS)
- Tokens em query string (ficam em logs)
- Debug info em stack traces de produção

**Ação**: Planejar para as próximas 1-2 sprints. Priorizar se outros achados amplificam o risco.

---

## 🔵 Baixo

**Quando usar**: Impacto mínimo isolado. Pode contribuir para postura de segurança fraca
mas não representa risco imediato.

**Impacto de produto**: Melhoria de postura, hardening, redução de superfície de ataque.

**Exemplos**:
- Versão de software exposta em headers
- Cookie sem `SameSite` em rota não-crítica
- Dependência desatualizada sem CVE conhecido
- Falta de HTTPS em ambiente de staging
- Comentários de código com informações internas

**Ação**: Incluir em backlog. Corrigir em janelas de manutenção ou junto a outras mudanças.

---

## ℹ️ Informativo

**Quando usar**: Observação de boa prática, melhoria de processo ou documentação.
Sem risco técnico imediato.

**Exemplos**:
- Ausência de audit log para ações não-críticas
- Falta de documentação de modelo de ameaças
- Melhoria de UX de segurança (ex: indicador de força de senha)
- Sugestão de ferramenta ou processo

**Ação**: Considerar como melhoria contínua. Sem urgência.

---

## Matriz de Decisão Rápida

| Exploração | Autenticação necessária | Dados afetados | Severidade |
|-----------|------------------------|----------------|-----------|
| Remota sem auth | Não | Todos os usuários | 🔴 Crítico |
| Remota sem auth | Não | Usuário atacante | 🟠 Alto |
| Requer auth | Sim | Outros usuários | 🟠 Alto |
| Requer auth | Sim | Próprio usuário | 🟡 Médio |
| Condicional complexa | Sim | Limitado | 🔵 Baixo |
| Informacional | N/A | Nenhum | ℹ️ Info |
