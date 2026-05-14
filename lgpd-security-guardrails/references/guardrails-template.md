# Agent Legal & Privacy Guardrails

> **Este arquivo é obrigatório.** Futuros agentes de IA devem lê-lo antes de tocar
> em auth, cadastro, checkout, pagamento, comunidade, suporte, termos, privacidade,
> consentimento, logs, migrations ou qualquer coleta de dados pessoais.

---

## 1. Segurança e Segredos

### Regras absolutas

- **Nunca** hardcodar tokens, API keys, senhas, cookies, private keys ou secrets no código
- **Nunca** imprimir ou logar `.env`, headers `Authorization`, cookies ou configs de providers
- **Nunca** gerar exemplos com credenciais reais — usar `YOUR_API_KEY`, `$ENV_VAR`, ou `***`
- Usar sempre variáveis de ambiente ou secret manager (ex: Vault, AWS Secrets Manager, Doppler)

### Logs

- Mascarar dados sensíveis antes de logar
- **Não logar:** body completo de requisições, headers completos, cookies, auth state, payloads com dados pessoais
- Ao logar erros de auth, usar apenas: user_id (hash), timestamp, tipo do erro — nunca o valor do token

### Checklist de segurança antes de qualquer PR

- [ ] Nenhum secret hardcoded (rodar `git-secrets` ou `trufflehog` se disponível)
- [ ] Nenhum `.env` commitado
- [ ] Logs filtrados sem dados pessoais ou auth
- [ ] Variáveis de ambiente documentadas em `.env.example` com valores fictícios

---

## 2. Dados Pessoais e LGPD

### Princípios

- **Finalidade**: não coletar dados pessoais sem finalidade clara e documentada
- **Minimização**: salvar apenas o mínimo necessário para a função
- **Transparência**: deixar claro na política de privacidade quais dados são coletados e por quê

### Responsabilidade

Este produto atua como **controlador** dos dados que opera diretamente (ex: dados de usuários cadastrados na plataforma).

O **usuário/cliente** é responsável pelos dados dos próprios clientes, projetos, prompts, deploys e bancos que ele gerencia dentro do produto. Esta plataforma não é responsável por dados de terceiros que o usuário introduz.

A atuação como **operador** (processar dados em nome de outro controlador) requer contrato/DPA específico — não assumir essa posição por padrão.

### Direitos dos titulares (Art. 18 LGPD)

Garantir mecanismos para:
- Acesso aos próprios dados
- Correção de dados incorretos
- Exclusão de dados (right to be forgotten)
- Portabilidade
- Revogação de consentimento

---

## 3. Aceite e Termos

Se o projeto tiver cadastro, checkout, comunidade ou qualquer feature SaaS:

### Regras de preservação

- **Não remover** checkbox de aceite de termos
- **Não remover** aviso de privacidade no cadastro ou checkout
- **Não remover** aviso de risco (se aplicável — ex: produtos financeiros, IA generativa)
- **Não remover** links para Termos de Uso e Política de Privacidade
- **Não prometer** "isenção total" de responsabilidade — preservar separação entre responsabilidade do produto e do usuário

### Texto mínimo de aceite recomendado

```
Ao continuar, você concorda com nossos [Termos de Uso] e [Política de Privacidade].
```

Qualquer alteração nesse texto deve ser revisada por responsável jurídico antes de ir para produção.

---

## 4. IP, User-Agent e Prova de Aceite

### Estrutura recomendada para registrar aceite de termos

```sql
CREATE TABLE term_acceptances (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     uuid NOT NULL REFERENCES auth.users(id),
  document_type    text NOT NULL,  -- 'terms_of_service' | 'privacy_policy'
  document_version text NOT NULL,  -- ex: '2024-01'
  accepted_at timestamp with time zone DEFAULT now(),
  source      text,                -- 'web' | 'mobile' | 'api'
  ip_hash     text,                -- HMAC-SHA256 do IP com salt secreto
  user_agent_hash text             -- HMAC-SHA256 do user-agent com salt secreto
);
```

### IP como dado pessoal

IP pode ser dado pessoal conforme LGPD. Por padrão:

- **Preferir hash com salt secreto** armazenado em variável de ambiente (`IP_HASH_SALT`)
- **Não salvar IP cru** sem: finalidade clara, retenção definida, acesso restrito e cobertura na política de privacidade
- Se IP cru for necessário (ex: segurança, fraud detection): documentar retenção máxima e acesso restrito

```typescript
// Correto — hash com salt antes de salvar
import { createHmac } from 'crypto';

function hashIp(ip: string): string {
  const salt = process.env.IP_HASH_SALT;
  if (!salt) throw new Error('IP_HASH_SALT não configurado');
  return createHmac('sha256', salt).update(ip).digest('hex');
}
```

---

## 5. Banco de Dados e RLS (Supabase/Postgres)

### Regra geral

**Toda tabela nova deve ter RLS habilitado na mesma migration em que é criada.**

### Template de migration segura

```sql
-- Criar tabela
CREATE TABLE minha_tabela (
  id    uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL REFERENCES auth.users(id),
  dado  text
);

-- OBRIGATÓRIO na mesma migration
ALTER TABLE minha_tabela ENABLE ROW LEVEL SECURITY;

-- Policies explícitas — nunca usar `USING (true)` sem justificativa forte
CREATE POLICY "usuarios veem apenas seus dados"
  ON minha_tabela FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "usuarios inserem apenas seus dados"
  ON minha_tabela FOR INSERT
  WITH CHECK (auth.uid() = user_id);
```

### Regras

- Limitar acesso por `auth.uid()`, ownership, papel interno ou `service_role`
- **Nunca usar** `USING (true)` ou `WITH CHECK (true)` sem justificativa documentada no próprio arquivo de migration
- Tabelas server-only/admin: manter RLS habilitado + documentar que o acesso ocorre via `service_role` em rotas protegidas
- Antes de concluir qualquer migration: verificar que nenhuma tabela ficou sem RLS

### Checklist de migration

- [ ] RLS habilitado em todas as tabelas novas
- [ ] Policies explícitas criadas (sem `true` genérico)
- [ ] Acesso via service_role documentado quando aplicável
- [ ] Migration testada em ambiente de desenvolvimento antes de produção

---

## 6. Comunidade e Suporte

Se o projeto tiver Discord, WhatsApp, fórum, chat de suporte ou qualquer canal de comunidade:

### Regras para usuários

Publicar avisos claros proibindo:
- Envio de tokens, API keys, `.env` ou credenciais
- Envio de logs com dados reais de clientes
- Prints sem mascarar dados pessoais (CPF, email, nome, endereço)
- Compartilhamento de dados de clientes de terceiros

### Regras para o produto/mantenedores

- Respostas de comunidade são **informais** e não constituem suporte oficial nem garantia legal
- Preservar poder de moderação e remoção de conteúdo inadequado
- Orientar usuários a mascarar dados antes de pedir ajuda (ex: `user_id: ***`, `email: u***@***.com`)

---

## 7. Checklist Final para Agentes de IA

Antes de submeter qualquer PR que envolva as áreas acima, confirme:

- [ ] Nenhum secret hardcoded ou exposto em output
- [ ] Logs filtrados — sem dados pessoais, tokens ou auth state
- [ ] Checkbox de aceite e links de termos preservados
- [ ] Toda tabela nova com RLS + policies explícitas
- [ ] IP salvo como hash (se salvo)
- [ ] Responsabilidades controlador/operador documentadas
- [ ] Direitos dos titulares endereçados (acesso, correção, exclusão)

---

*Última revisão: {{DATA}}*
*Responsável: {{RESPONSAVEL}}*
*Versão: 1.0*
