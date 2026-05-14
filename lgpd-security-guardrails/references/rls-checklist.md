# RLS Checklist — Supabase / Postgres

## Verificar tabelas sem RLS habilitado

```sql
-- Executar no SQL Editor do Supabase ou psql
SELECT
  schemaname,
  tablename,
  rowsecurity
FROM pg_tables
WHERE schemaname = 'public'
  AND rowsecurity = false
ORDER BY tablename;
```

Qualquer linha retornada = tabela sem RLS = **problema crítico de segurança**.

---

## Verificar policies existentes

```sql
-- Ver todas as policies do schema public
SELECT
  schemaname,
  tablename,
  policyname,
  permissive,
  roles,
  cmd,
  qual,
  with_check
FROM pg_policies
WHERE schemaname = 'public'
ORDER BY tablename, policyname;
```

Checar se alguma policy tem `qual = 'true'` ou `with_check = 'true'` — se sim, documentar justificativa.

---

## Padrões de policies por tipo de acesso

### Dados do próprio usuário (user-owned)

```sql
ALTER TABLE tabela ENABLE ROW LEVEL SECURITY;

CREATE POLICY "select_own" ON tabela FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "insert_own" ON tabela FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "update_own" ON tabela FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "delete_own" ON tabela FOR DELETE
  USING (auth.uid() = user_id);
```

### Dados públicos (somente leitura)

```sql
ALTER TABLE tabela ENABLE ROW LEVEL SECURITY;

CREATE POLICY "select_public" ON tabela FOR SELECT
  USING (true);  -- Justificativa: dados públicos, sem PII

-- INSERT/UPDATE/DELETE apenas via service_role (sem policy = bloqueado para usuários)
```

### Tabela admin/server-only

```sql
ALTER TABLE tabela ENABLE ROW LEVEL SECURITY;
-- Sem policies = nenhum usuário autenticado acessa
-- Acesso somente via service_role em rotas protegidas do backend
-- Documentar: "Acesso via service_role em /api/admin/*"
```

### Acesso por papel interno (role-based)

```sql
ALTER TABLE tabela ENABLE ROW LEVEL SECURITY;

CREATE POLICY "select_admin_role" ON tabela FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM user_roles
      WHERE user_id = auth.uid()
        AND role = 'admin'
    )
  );
```

---

## Armadilhas comuns

| Erro | Sintoma | Correção |
|------|---------|----------|
| RLS não habilitado | Qualquer usuário autenticado lê tudo | `ALTER TABLE x ENABLE ROW LEVEL SECURITY` |
| `USING (true)` sem intenção | Dados de outros usuários visíveis | Substituir por `auth.uid() = user_id` |
| Sem policy de INSERT | Usuário não consegue inserir | Adicionar `WITH CHECK` correta |
| service_role bypassa RLS | Comportamento inesperado em testes | Normal — service_role sempre bypassa, testar com anon/authenticated |
| RLS em tabelas de referência | Performance | Considerar `security definer` functions para lookups frequentes |

---

## Teste rápido de RLS

Após criar a migration, testar com usuário autenticado diferente do owner:

```sql
-- Simular usuário específico no SQL Editor do Supabase
SET LOCAL role authenticated;
SET LOCAL request.jwt.claims = '{"sub": "outro-user-id"}';

SELECT * FROM tabela WHERE user_id = 'user-id-real';
-- Deve retornar 0 linhas
```
