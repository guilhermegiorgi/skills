# Domínios de Segurança — AppSec Audit

Para cada domínio, o que procurar ativamente no projeto.

---

## 1. Autenticação e Sessões

- Tokens de sessão com expiração configurada?
- Refresh token rotation implementado?
- Logout invalida sessão server-side ou apenas client-side?
- Brute-force protection (rate limit em `/login`, `/signup`, `/reset`)?
- MFA disponível ou obrigatório para operações sensíveis?
- Senhas com hash seguro (bcrypt, argon2, scrypt)? Nunca MD5/SHA1 simples
- JWT: algoritmo `none` aceito? Chave fraca? Claims validados?
- Cookie flags: `Secure`, `HttpOnly`, `SameSite=Strict/Lax`?
- Session fixation possível?

## 2. Autorização, Permissões e Multi-tenant

- IDOR (Insecure Direct Object Reference): usuário A consegue acessar recursos do usuário B por ID?
- RLS habilitado em todas as tabelas do banco?
- Verificação de ownership em cada query ou apenas na rota?
- Endpoints admin acessíveis sem role check?
- Isolamento entre workspaces/tenants/organizações?
- Escalada de privilégio possível (ex: campo `role` editável pelo próprio usuário)?
- Permissões verificadas no backend ou apenas no frontend?

## 3. Exposição de Tokens, API Keys e Credenciais

- Secrets hardcoded no código-fonte?
- `.env` ou arquivos de config commitados no git?
- Tokens em logs, stack traces ou respostas de erro?
- API keys expostas em bundle JS do frontend?
- URLs assinadas com expiração longa ou sem controle?
- Tokens em query string (ficam em logs de servidor/proxy)?
- `git log` ou histórico com credenciais antigas?

## 4. Dependências Maliciosas ou Vulneráveis

- Dependências com CVEs conhecidos (npm audit, pip-audit, cargo audit)?
- Typosquatting: nomes de pacotes suspeitos similares a populares?
- Dependências abandonadas (última publicação > 2 anos, 0 mantenedores)?
- Uso de `*` ou ranges muito amplos em versões?
- Post-install scripts suspeitos em `package.json`?
- Pacotes com acesso excessivo a filesystem ou rede sem justificativa?

## 5. Scripts de Instalação Perigosos

- `preinstall`, `postinstall`, `prepare` scripts que baixam binários externos?
- `curl | bash` ou equivalente no setup?
- Scripts que leem variáveis de ambiente do sistema durante instalação?
- Makefile ou scripts de CI que executam código não-auditado?
- Docker entrypoint com permissões excessivas?

## 6. Uso Inseguro de Terminal, Shell e Execução de Comandos

- `exec()`, `spawn()`, `system()`, `subprocess` com input não sanitizado?
- Shell injection possível via interpolação de strings em comandos?
- Comandos construídos com dados do usuário sem escapamento?
- Uso de `eval()` com conteúdo dinâmico?
- Permissões de processo mais amplas que o necessário (root desnecessário)?

## 7. Leitura/Escrita Insegura de Arquivos

- Caminhos de arquivo construídos com input do usuário sem validação?
- Escrita em diretórios temporários compartilhados sem nonce único?
- Race condition em operações de arquivo (TOCTOU)?
- Permissões de arquivo criadas como world-readable?
- Arquivos temporários com dados sensíveis não deletados após uso?

## 8. Path Traversal, Symlinks e Escape de Diretórios

- Input `../../../etc/passwd` sanitizado em rotas de arquivo?
- Symlinks seguidos além do diretório permitido?
- `realpath()` ou equivalente usado antes de operações em arquivo?
- Extração de arquivos ZIP/TAR valida destino dos paths internos (zip slip)?

## 9. SSRF e Chamadas de Rede para Destinos Não Confiáveis

- Endpoints que fazem fetch/request para URL fornecida pelo usuário?
- Allowlist de destinos de rede implementada?
- Redirecionamentos HTTP seguidos sem validação?
- Metadata de cloud (169.254.169.254) acessível via SSRF?
- DNS rebinding mitigado?
- Webhooks validam origem (HMAC signature)?

## 10. Upload e Download de Arquivos

- Tipo MIME validado server-side (não apenas client-side)?
- Conteúdo do arquivo inspecionado além da extensão?
- Uploads servidos do mesmo domínio da aplicação (risco de XSS via SVG/HTML)?
- Nome do arquivo sanitizado antes de salvar no disco?
- Limite de tamanho e rate limit em uploads?
- Downloads de arquivos gerados validam ownership?

## 11. Logs, Histórico e Vazamento Acidental

- Logs incluem tokens, senhas, CPF, cartão, dados de saúde?
- Stack traces com dados sensíveis expostos para o usuário final?
- Histórico de queries/prompts do usuário protegido?
- Logs de debug ativos em produção?
- Error tracking (Sentry, Datadog) capturando PII?
- Respostas de erro da API revelam stack interno ou estrutura do banco?

## 12. Banco de Dados, Queries e Injeção

- Queries construídas com interpolação de strings (SQL injection)?
- ORM usado corretamente (sem `.raw()` com input não sanitizado)?
- NoSQL injection possível (MongoDB `$where`, `$regex`)?
- Stored procedures com permissões excessivas?
- Backups de banco criptografados?
- Connection strings com credenciais hardcoded?

## 13. XSS, CSRF e Injeção no Frontend

- Output de dados do usuário escapado antes de renderizar no HTML?
- `dangerouslySetInnerHTML` ou `v-html` usado com conteúdo dinâmico?
- CSP (Content Security Policy) configurado?
- CSRF tokens em formulários e mutations?
- `SameSite` cookie attribute configurado?
- `postMessage` com `targetOrigin: *`?
- DOM-based XSS em `location.hash`, `document.write`, `innerHTML`?

## 14. Prompt Injection e Abuso de Agentes/LLMs

- Input do usuário inserido diretamente em prompts do sistema?
- Separação clara entre instruções do sistema e dados do usuário no prompt?
- Agentes LLM com acesso a ferramentas críticas (delete, send, publish) sem confirmação?
- Output do LLM renderizado como HTML sem sanitização?
- Tool calls do LLM validam permissões do usuário atual?
- Jailbreak protection implementada?
- Limites de tokens/custo por usuário/tenant?

## 15. Supply Chain, Lockfile e Assinatura de Releases

- Lockfile (`package-lock.json`, `yarn.lock`, etc.) commitado e verificado?
- `npm install` sem `--frozen-lockfile` em CI?
- Checksums ou SBOMs gerados para releases?
- Dependências de desenvolvimento isoladas das de produção?
- Pacotes instalados de registries privados sem autenticação?
- GitHub Actions usa tags de commit fixas ou `@latest` dinâmico?

## 16. Configurações de Produção, Debug e CORS

- `DEBUG=true` ou equivalente em produção?
- CORS com `Allow-Origin: *` em APIs autenticadas?
- Headers de segurança presentes? (`X-Frame-Options`, `X-Content-Type-Options`, `HSTS`, `Permissions-Policy`)
- Stack traces visíveis em respostas HTTP 500?
- Swagger/OpenAPI ou admin panel exposto em produção sem auth?
- Rate limiting e throttling configurados?
- HTTPS forçado em todas as rotas?

## 17. Isolamento entre Projetos, Workspaces e Ambientes

- Dados de produção acessíveis em ambiente de staging?
- Chaves/tokens compartilhados entre ambientes?
- Feature flags de desenvolvimento acessíveis em produção?
- Isolamento de recursos por workspace/tenant em serviços externos (storage, queue, cache)?

## 18. Auditoria, Rastreabilidade e Alertas

- Audit log de ações sensíveis (login, delete, export, admin actions)?
- Alertas para anomalias (múltiplos logins, acesso fora do padrão, exports em massa)?
- Logs imutáveis ou protegidos contra alteração pelo próprio sistema?
- Correlação de eventos por `user_id` + `session_id` + `request_id`?
- Retenção de logs definida e aplicada?

## 19. Exposição de Informações e Fingerprinting

- Headers de servidor revelam versão (ex: `Server: nginx/1.18.0`)?
- Enumeração de usuários possível via mensagens de erro distintas?
- Timing attacks possíveis em comparação de tokens?
- Metadados de arquivos (EXIF, autor de documento) expostos em uploads?

## 20. Proteção de Dados Pessoais e Compliance

- PII armazenada além do necessário?
- Dados pessoais criptografados em repouso?
- Consentimento registrado antes de coleta?
- Direito de exclusão implementado (hard delete ou pseudonimização)?
- Dados de menores tratados com proteção adicional?
