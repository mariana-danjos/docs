# ADR — Documento de Decisão de Arquitetura
### Plus Auth — MS Auth + MFE de Autenticação

| Campo | Valor |
|---|---|
| **Projeto** | Sistema de gestão de estoque para loja de roupas plus size |
| **Disciplina** | Engenharia de Software II — 98802-02 |
| **Turma** | 30 — PUCRS / 2026-1 |
| **Professor** | Prof. José Pedro Schardosim Simão |
| **Alunas** | Gabriela Menegaz, Julia Kriedte, Lívia Noer e Mariana Adam |
| **Versão / Data** | v1.1 — Maio de 2026 |
| **Status** | ✔ Aceita |

---

## 1. Contexto e Problema

Serviço responsável pela identidade dos usuários do sistema Plus. Deve suportar cadastro, login, emissão e renovação de JWT, controle de acesso por papel (RBAC) e revogação de tokens. O microfrontend de autenticação é consumido pelo Shell App via Module Federation e expõe o contexto de sessão para os demais MFEs.

Este ADR documenta as decisões sobre linguagem, framework, estratégia de tokens, banco de dados, design do MFE e integração com o Shell.

---

## 2. Decisão Arquitetural

Implementar o MS Auth em Node.js 20 + TypeScript + Express, com JWT (access 15 min + refresh 7 dias), RBAC por roles, persistência em PostgreSQL e documentação OpenAPI em `/api-docs`. O MFE de autenticação é construído com React 18 + Vite 5 + MUI 9, expondo `LoginPage`, `SignupPage`, `AuthContext` e `theme` via Module Federation.

> **Decisão Central:** MS Auth stateless em Express + TypeScript, tokens JWT com blocklist em PostgreSQL, RBAC com 3 roles (admin / vendedor / gestor). MFE como remote Module Federation (Vite), expondo contexto de auth compartilhado para o Shell e demais MFEs. Ambiente local via Docker Compose + Ministack.

---

## 3. Responsabilidades do MS Auth

| Responsabilidade | Detalhe |
|---|---|
| **Cadastro** | `POST /auth/signup` (alias: `/auth/register`) — cria usuário, retorna access + refresh token |
| **Login** | `POST /auth/login` — autentica, retorna access + refresh token |
| **Renovação** | `POST /auth/refresh` — emite novo access token a partir do refresh token |
| **Logout** | `POST /auth/logout` — revoga o access token (blocklist) e exclui refresh tokens |
| **Perfil** | `GET /auth/me` — retorna dados do usuário autenticado |
| **Roles disponíveis** | `GET /auth/roles` — lista os papéis válidos do sistema |
| **RBAC** | `POST/DELETE /auth/users/:id/roles` — atribui e remove roles (admin only) |
| **Documentação** | `GET /api-docs` — Swagger UI com a especificação OpenAPI 3.0 |

---

## 4. Visão da Arquitetura (T1)

### 4.1 Fluxo Geral

O browser carrega o **Shell App** (localhost:3000), que consome o `plus-mfe-auth` via Module Federation. O MFE expõe as páginas de login/cadastro e o `AuthContext`, que persiste o token em `localStorage` e agenda o refresh automático 60 s antes da expiração.

As chamadas à API passam pelo **MS Auth** (localhost:3001). Em ambiente local, o banco PostgreSQL é provisionado pelo **Ministack** via Terraform. Os demais microsserviços (T2) receberão os claims do JWT via header, sem revalidar o token.

### 4.2 Camadas

| Camada | Tecnologia | Descrição |
|---|---|---|
| **Shell App** | React + Vite (localhost:3000) | Host do Module Federation; roteamento e contexto de auth |
| **MFE Auth** | React 18 + MUI 9 + Vite (localhost:4001) | Remote MF: LoginPage, SignupPage, AuthContext, theme |
| **MS Auth** | Node.js 20 + Express (localhost:3001) | API REST de autenticação; emite e revoga JWTs |
| **Banco de dados** | PostgreSQL (localhost:5432) | users, refresh_tokens, token_blocklist |
| **Dev local** | Docker Compose + Ministack | Emula RDS, S3, API Gateway e STS sem conta AWS; provisionado via Terraform |

---

## 5. Tecnologias Adotadas

### 5.1 Microsserviço

| Pacote | Finalidade |
|---|---|
| Node.js 20 LTS + TypeScript 5 | Runtime + tipagem estática |
| Express 4 | Framework HTTP |
| `jsonwebtoken` 9 | Emissão e verificação de JWT (HS256) |
| `bcryptjs` 2 (12 rounds) | Hash de senhas |
| `pg` 8 | Cliente PostgreSQL (pool) |
| `zod` 3 | Validação de schemas (body, params, headers) |
| `express-rate-limit` 7 | Rate limiting por rota (signup: 5/min por IP; login: 10/min por e-mail) |
| `pino` 9 | Logger estruturado (JSON) |
| `swagger-jsdoc` + `swagger-ui-express` | OpenAPI 3.0 em `/api-docs` |
| `node-pg-migrate` 7 | Migrations SQL versionadas em TypeScript |
| Jest + Supertest | Testes de integração (~97% de cobertura) |

### 5.2 Microfrontend

| Pacote | Finalidade |
|---|---|
| React 18 + TypeScript + Vite 5 | UI + tipagem + bundler |
| `@originjs/vite-plugin-federation` 1.3 | Module Federation (remote) |
| MUI 9 + Emotion | Design system obrigatório |
| `react-hook-form` + Zod 4 | Formulários com validação |
| Vitest + Testing Library + MSW | Testes unitários (~84% de cobertura) |

---

## 6. Segurança

| Mecanismo | Detalhe |
|---|---|
| **JWT** | HS256. Access token 15 min · Refresh token 7 dias. Secrets em variáveis de ambiente. |
| **Senhas** | `bcryptjs` 12 rounds. |
| **Timing attack** | Login sempre executa `bcrypt.compare` com hash dummy quando e-mail não existe. |
| **Revogação** | Access tokens → `token_blocklist` (SHA-256 + TTL). Refresh tokens → tabela própria, deletados no logout. |
| **Hash de tokens** | Refresh tokens e entradas da blocklist armazenam SHA-256 do JWT, nunca o token cru. |
| **Validação** | Zod em todos os inputs. Sanitização de caracteres de controle e HTML escape antes da persistência. |
| **Rate limit** | Signup: 5 req/min por IP. Login: 10 req/min por e-mail. Desabilitado em `NODE_ENV=test`. |
| **CORS** | Whitelist `localhost:3000` e `localhost:4001`. Credentials habilitado. |

---

## 7. Alternativas Consideradas

| Alternativa | Prós | Contras / Motivo da Rejeição |
|---|---|---|
| Python (FastAPI) | Sintaxe concisa; bom suporte a async | Menor familiaridade da equipe; ecossistema TypeScript já adotado no projeto |
| NestJS (Node.js) | Estrutura opinada, DI, decorators | Overhead desnecessário para o escopo; curva de aprendizado maior sem ganho proporcional |
| Token único de longa duração | Implementação mais simples | Janela de exposição grande; revogação exige blocklist de qualquer forma |
| Token opaco (session ID no banco) | Revogação imediata | Exige lookup no banco a cada requisição; não é stateless; incompatível com API Gateway |
| Webpack 5 Module Federation | Mais maduro; maior documentação | Build e HMR mais lentos; configuração mais verbosa que Vite |
| Iframe (MFE sem Module Federation) | Isolamento total | UX degradada; sem compartilhamento de estado nem design system |
| Redux / Zustand para estado de auth | Mais robusto para estado complexo | Escopo restrito a sessão de autenticação; React Context é suficiente |
| Serviço externo (Auth0, Keycloak) | Pronto para produção; alta segurança | Não cumpre o requisito pedagógico de implementar o MS Auth |

---

## 8. Consequências

### 8.1 Positivas

- **Segurança robusta:** bcrypt 12 rounds, tokens de curta duração, blocklist, hash SHA-256 dos tokens no banco e proteção contra timing attack
- **Contrato claro:** OpenAPI em `/api-docs` facilita a integração pelos demais grupos na T2
- **Contexto federado:** `AuthContext` e `theme` expostos via Module Federation evitam duplicação de lógica e estilo nos outros MFEs
- **Ambiente reproduzível:** Docker Compose + Ministack + Terraform garantem paridade entre máquinas de desenvolvimento

### 8.2 Negativas / Riscos
- **Refresh token sem rotação:** o token é reutilizado até expirar; a rotação a cada uso aumentaria a segurança contra roubo de token

---

## 9. Histórico de Revisões

| Versão | Data | Autor | Descrição |
|---|---|---|---|
| v1.0 | Mai/2026 | Equipe | Versão inicial — decisões do T1 (MS Auth + MFE Auth) |
| v1.1 | Mai/2026 | Equipe | Inclusão da seção de segurança e ajuste das alternativas consideradas |
