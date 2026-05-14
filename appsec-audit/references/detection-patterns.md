# Padrões de Detecção Rápida — AppSec Audit

Execute estes comandos durante a fase de mapeamento. Filtrar saída para não expor valores reais.

## Secrets hardcoded

```bash
# Procurar padrões de secrets (reportar APENAS o arquivo e linha, nunca o valor)
grep -rn \
  -e "api_key\s*=\s*['\"]" \
  -e "secret\s*=\s*['\"]" \
  -e "password\s*=\s*['\"]" \
  -e "token\s*=\s*['\"]" \
  -e "private_key\s*=" \
  --include="*.ts" --include="*.js" --include="*.py" --include="*.go" \
  --include="*.env*" --include="*.yml" --include="*.yaml" \
  . 2>/dev/null \
  | grep -v ".env.example" \
  | grep -v "YOUR_" \
  | grep -v "REPLACE_" \
  | sed 's/=.*/= ***/' \
  | head -30
```

## SQL injection candidates

```bash
grep -rn \
  -e "query.*\`\${" \
  -e "query.*+.*req\." \
  -e "\.raw(" \
  -e "execute.*f\"" \
  -e "cursor\.execute.*%" \
  --include="*.ts" --include="*.js" --include="*.py" . 2>/dev/null | head -20
```

## Shell injection candidates

```bash
grep -rn \
  -e "exec(" \
  -e "spawn(" \
  -e "execSync(" \
  -e "subprocess\.run" \
  -e "os\.system(" \
  -e "shell=True" \
  --include="*.ts" --include="*.js" --include="*.py" . 2>/dev/null | head -20
```

## eval() usage

```bash
grep -rn "eval(" \
  --include="*.ts" --include="*.js" --include="*.py" . 2>/dev/null \
  | grep -v "evalId\|evalMetric\|evaluate\|//.*eval" | head -15
```

## XSS candidates

```bash
grep -rn \
  -e "dangerouslySetInnerHTML" \
  -e "v-html=" \
  -e "innerHTML\s*=" \
  -e "document\.write(" \
  -e "\.insertAdjacentHTML(" \
  --include="*.tsx" --include="*.jsx" --include="*.vue" --include="*.html" \
  . 2>/dev/null | head -20
```

## SSRF candidates

```bash
grep -rn \
  -e "fetch(req\." \
  -e "axios\.get(req\." \
  -e "fetch(body\." \
  -e "requests\.get(url" \
  -e "httpx.*url.*request" \
  --include="*.ts" --include="*.js" --include="*.py" . 2>/dev/null | head -20
```

## Path traversal candidates

```bash
grep -rn \
  -e "path\.join.*req\." \
  -e "readFile.*req\." \
  -e "fs\.read.*params" \
  -e "open(.*request\." \
  --include="*.ts" --include="*.js" --include="*.py" . 2>/dev/null | head -20
```

## Prompt injection candidates

```bash
grep -rn \
  -e "messages.*role.*user.*content.*input" \
  -e "prompt.*\${.*user" \
  -e "system.*\${" \
  -e "template.*inject" \
  --include="*.ts" --include="*.js" --include="*.py" . 2>/dev/null | head -20
```

## CORS configuration

```bash
grep -rn \
  -e "Access-Control-Allow-Origin.*\*" \
  -e "cors.*origin.*true" \
  -e "allowedOrigins.*\*" \
  --include="*.ts" --include="*.js" --include="*.py" \
  --include="*.json" --include="*.yml" . 2>/dev/null | head -15
```

## Debug flags em produção

```bash
grep -rn \
  -e "DEBUG.*=.*true" \
  -e "NODE_ENV.*development" \
  -e "FLASK_DEBUG.*1" \
  -e "debug.*mode.*true" \
  --include="*.ts" --include="*.js" --include="*.py" \
  --include="*.json" --include="*.yml" --include="*.env*" \
  . 2>/dev/null | grep -v ".env.example" | head -15
```

## Dependências — auditoria rápida

```bash
# Node.js
npm audit --json 2>/dev/null | python3 -c "
import json,sys
d=json.load(sys.stdin)
vulns=d.get('vulnerabilities',{})
critical=[k for k,v in vulns.items() if v.get('severity') in ['critical','high']]
print(f'Critical/High vulnerabilities: {len(critical)}')
for c in critical[:10]: print(f'  - {c}: {vulns[c][\"severity\"]}')
" 2>/dev/null

# Python
pip-audit --format=json 2>/dev/null | python3 -c "
import json,sys
data=json.load(sys.stdin)
vulns=data.get('dependencies',[])
for d in vulns:
  for v in d.get('vulns',[]):
    print(f'{d[\"name\"]} {d[\"version\"]}: {v[\"id\"]} ({v.get(\"fix_versions\",\"no fix\")})')
" 2>/dev/null | head -20
```

## Lockfile presente?

```bash
echo "=== Lockfiles ==="
ls -la package-lock.json yarn.lock pnpm-lock.yaml bun.lockb poetry.lock \
  Pipfile.lock Cargo.lock go.sum 2>/dev/null

echo "=== .gitignore inclui .env? ==="
grep "\.env" .gitignore 2>/dev/null || echo "ATENÇÃO: .env não está no .gitignore"

echo "=== Arquivos .env no git? ==="
git ls-files | grep -E "\.env$|\.env\." 2>/dev/null || echo "Nenhum .env no git (OK)"
```

## RLS habilitado? (Supabase)

```bash
# Procurar migrations sem RLS
grep -rn "CREATE TABLE" --include="*.sql" . 2>/dev/null \
  | while IFS=: read file line content; do
      table=$(echo "$content" | grep -o 'TABLE [^ (]*' | awk '{print $2}')
      file_content=$(cat "$file" 2>/dev/null)
      if ! echo "$file_content" | grep -qi "ENABLE ROW LEVEL SECURITY\|enable row level security"; then
        echo "⚠️  $file: tabela '$table' pode não ter RLS"
      fi
    done
```

## JWT fraco

```bash
grep -rn \
  -e "algorithm.*none\|alg.*none" \
  -e "jwt\.sign.*'secret'\|jwt\.sign.*\"secret\"" \
  -e "verify.*ignoreExpiration.*true" \
  --include="*.ts" --include="*.js" . 2>/dev/null | head -10
```

## Tokens em query string

```bash
grep -rn \
  -e "?\(token\|api_key\|apikey\|access_token\|auth\)=" \
  -e "searchParams.*token\|query.*token" \
  --include="*.ts" --include="*.js" . 2>/dev/null | head -10
```
