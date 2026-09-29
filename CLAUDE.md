# SecVault — Contexto do Projeto

> Handoff para o Claude Code. Este arquivo é **autossuficiente**: traz o blueprint do projeto e,
> no fim, a base de segurança e as convenções do portfólio. A central do portfólio é
> [Everett-gi/Projetos-e-ideias](https://github.com/Everett-gi/Projetos-e-ideias). Referências de qualidade:
> [DocSage](https://github.com/Everett-gi/docsage) e [PwnCheck](https://github.com/Everett-gi/pwncheck)
> (estrutura, testes, CI e as lições do modo tutorial).
> **Status:** 📋 blueprint pronto — a construção começa pela fase 1.

## Modo tutorial

O autor programa bem em **C e C++** (lógica, ponteiros, memória, compilação) e
está aprendendo Python; **Java é novo para ele** — este projeto também é o curso de Java. Se outro projeto Java do portfólio já tiver ensinado o básico, não repita: aponte para a lição de lá.

- Explique cada passo e o **porquê**, com analogias a C/C++ quando ajudarem (ex.: JVM × compilação nativa, referências × ponteiros, garbage collector × RAII e `try-with-resources`, interfaces × classes abstratas puras, generics × templates, Maven × CMake + gerenciador de pacotes, anotações do Spring × atributos/macros).
  Não explique lógica de programação básica; foque no que é novo.
- Cada fase concluída ganha uma lição em `docs/tutorial/fase-N-<tema>.md`, com exercícios e
  respostas no fim — o modelo são as lições do PwnCheck.
- Confirme rodando código qualquer afirmação técnica antes de escrevê-la numa lição.
- Sessões na nuvem rodam em Linux (bash). Ao indicar comandos para o autor rodar na máquina
  dele, use PowerShell (Windows 11, VS Code, repositórios clonados em `C:\dev`).
- Commits no padrão Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `ci:`, `chore:`),
  um assunto por commit. Rode lint e testes sobre o estado final, logo antes de cada commit.

## Prioridades

**1) Segurança · 2) Clareza do código · 3) Funcionalidade.** Nessa ordem.

## O que é

Cofre para guardar segredos/senhas com **criptografia autenticada no servidor**, organizados
por workspaces, com master password e trilha de acessos.

**Valor de portfólio:** criptografia aplicada de verdade (AES-GCM, derivação de chave) e
modelagem de segurança — peça central de um portfólio de cibersegurança.

## Stack

Java 21 · Spring Boot 3.3 · Spring Security · Java Cryptography (AES-GCM) · Spring Data JPA ·
Flyway · PostgreSQL 16 · Maven · JUnit 5.

> **Versões:** o blueprint cita Spring Boot 3.3, cuja linha já saiu do suporte gratuito.
> No scaffold, use as versões estáveis atuais do Spring Boot e do JDK LTS (confira no
> start.spring.io) e registre a escolha na lição da fase 1. Segurança vem antes de fidelidade ao blueprint.

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
6. Trilha de auditoria + deploy (ver `docs/DEPLOY.md`)

## Como começar

1. Gere o esqueleto no [Spring Initializr](https://start.spring.io): projeto **Maven**, Java,
   dependências Spring Web, Spring Security, Spring Data JPA, Flyway, PostgreSQL Driver,
   Validation e Testcontainers. Extraia na **raiz deste repositório**, preservando `README.md`,
   `CLAUDE.md`, `.gitignore` e `.gitattributes` daqui (junte ao nosso `.gitignore` as regras
   do gerado).
2. Crie já na fase 1 o CI em `.github/workflows/ci.yml` (build e testes com `./mvnw verify`
   e o scan OWASP Dependency-Check), um `docker-compose.yml` com PostgreSQL 16 para
   desenvolvimento e um `.env.example` com valores fictícios.
3. **Comece pela camada de criptografia**, isolada e testada: cifra/decifra, falha ao
   decifrar um dado adulterado (é o "autenticada" do AES-GCM) e nonce único a cada
   gravação. É o coração do projeto. Use só as APIs padrão da JCA (`javax.crypto`) —
   nunca implemente criptografia à mão.
4. Mantenha a lógica de domínio testável sem banco (testes unitários rápidos) e use
   Testcontainers nos testes de integração. Se o ambiente não tiver Docker, os testes
   unitários precisam continuar rodando.

## Base de segurança do portfólio (Definition of Done)

Vale para todos os projetos do portfólio, este incluído:

- **Segredos** só em variáveis de ambiente (`.env` fora do Git e da imagem). Nunca hardcoded.
- **Senhas** com hash Argon2/BCrypt. **Nunca** texto puro ou hash fraco.
- **Autenticação** por JWT (access + refresh com rotação) onde houver login.
- **Autorização** com verificação de posse por recurso (anti-IDOR) e RBAC quando aplicável.
- **SQL Injection**: acesso ao banco só via ORM / parâmetros ligados. Nunca concatenar SQL.
- **Validação de entrada** com schema/DTO e limite de tamanho de payload.
- **Rate limiting** em login e endpoints sensíveis.
- **Cabeçalhos de segurança** (HSTS, CSP, X-Content-Type-Options) e **HTTPS** em produção.
- **Dependências** escaneadas no CI + Dependabot.
- **Logs** sem segredos nem dados pessoais.
- **LGPD**: coleta mínima, aceite de termos e exclusão de conta quando houver dados de usuário.
- Um **`SECURITY.md`** descrevendo essas medidas aplicadas a este repositório (crie-o na
  primeira fase com código e mantenha-o atualizado).

## Convenções de código

- Comentários e documentação do código em **português**; nomes de código em **inglês**.
- Tipos explícitos sempre (em Python, *type hints*).
- Toda função pura nova vem acompanhada de teste.
- Mensagens de erro da API/interface em português (é o usuário que lê).
- Ao criar uma variável de ambiente nova, atualize o `.env.example` (valores fictícios).

## Ferramentas — Java (Spring Boot)

| | |
|---|---|
| Migrations | Flyway |
| Auth | Spring Security |
| Testes | JUnit 5 + Testcontainers |
| Lint | Checkstyle / SpotBugs |
| Scan de segurança | OWASP Dependency-Check |

## Deploy

Guia em `docs/DEPLOY.md` (porta interna 8080).
