# SecVault — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
Cofre para guardar segredos/senhas com **criptografia autenticada no servidor**, organizados
por workspaces, com master password e trilha de acessos.

**Valor de portfólio:** criptografia aplicada de verdade (AES-GCM, derivação de chave) e
modelagem de segurança — peça central de um portfólio de cibersegurança.

## Stack
Java 21 · Spring Boot 3.3 · Spring Security · Java Cryptography (AES-GCM) · Spring Data JPA ·
Flyway · PostgreSQL 16 · Maven · JUnit 5.

## Modelo de dados
- **users** (id, email, master_key_hash [Argon2])
- **workspaces** (id, owner_id, name) + **workspace_members** (workspace_id, user_id, role)
- **secrets** (id, workspace_id, name, ciphertext, nonce, version, updated_at)
- **secret_versions** (id, secret_id, ciphertext, nonce, version, created_at)
- **access_log** (id, secret_id, user_id, action, created_at)

## Endpoints principais
- CRUD de segredos (sempre cifrados) · listagem por workspace
- Compartilhamento por workspace · rotação de segredo · histórico de versões
- Trilha de auditoria de acessos

## Foco de segurança
**AES-GCM** (criptografia autenticada) com **envelope encryption**; chave derivada da master
password via **Argon2/PBKDF2**; segredo **nunca** trafega/loga em claro; versionamento;
audit log de cada acesso; RBAC por workspace.

## Plano de build
1. Schema + migrations
2. Camada de criptografia isolada e testada (cifra/decifra, nonce único por gravação)
3. Autenticação + derivação da chave a partir da master password
4. CRUD de segredos cifrados + RBAC por workspace
5. Versionamento + rotação
6. Trilha de auditoria + deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.java`). Comece pela camada
de criptografia com testes cobrindo cifra/decifra e unicidade de nonce — é o coração do projeto.
