# SecVault

Cofre de segredos com criptografia autenticada (AES-GCM), workspaces, versionamento e trilha de auditoria.

> 📋 **Em construção.** O blueprint está pronto e a implementação começa pela fase 1 do
> roadmap abaixo. Parte do portfólio
> [Projetos-e-ideias](https://github.com/Everett-gi/Projetos-e-ideias).

## O que é

Cofre para guardar segredos/senhas com **criptografia autenticada no servidor**, organizados
por workspaces, com master password e trilha de acessos.

## Stack (planejada)

Java 21 · Spring Boot 3.3 · Spring Security · Java Cryptography (AES-GCM) · Spring Data JPA ·
Flyway · PostgreSQL 16 · Maven · JUnit 5.

## Modelo de dados (planejado)

- **users** (id, email, master_key_hash [Argon2])
- **workspaces** (id, owner_id, name) + **workspace_members** (workspace_id, user_id, role)
- **secrets** (id, workspace_id, name, ciphertext, nonce, version, updated_at)
- **secret_versions** (id, secret_id, ciphertext, nonce, version, created_at)
- **access_log** (id, secret_id, user_id, action, created_at)

## Endpoints principais

- CRUD de segredos (sempre cifrados) · listagem por workspace
- Compartilhamento por workspace · rotação de segredo · histórico de versões
- Trilha de auditoria de acessos

## Segurança

**AES-GCM** (criptografia autenticada) com **envelope encryption**; chave derivada da master
password via **Argon2/PBKDF2**; segredo **nunca** trafega/loga em claro; versionamento;
audit log de cada acesso; RBAC por workspace.

## Roadmap

1. Schema + migrations
2. Camada de criptografia isolada e testada (cifra/decifra, nonce único por gravação)
3. Autenticação + derivação da chave a partir da master password
4. CRUD de segredos cifrados + RBAC por workspace
5. Versionamento + rotação
6. Trilha de auditoria + deploy

---

Desenvolvido em modo tutorial: cada fase concluída ganha uma lição em `docs/tutorial/`.
