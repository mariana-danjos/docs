# Entregas

O projeto é dividido em duas entregas, cada uma composta por um microsserviço e seu respectivo microfrontend.

---

## T1 — Autenticação e Autorização

Desenvolvimento do **MS Auth** e do **MFE de autenticação**.

O microsserviço deverá suportar:

- Cadastro e login de usuários
- Emissão e refresh de JWT
- RBAC (controle de permissões por role)
- Revogação de tokens

O microfrontend deverá incluir as telas de login/logout e o gerenciamento de sessão integrado ao Shell App.

Após a entrega, a turma vota o melhor par MS Auth + MFE, que será adotado como solução oficial na T2.

---

## T2 — Domínio

Desenvolvimento de um **MS de domínio** e seu **MFE**, utilizando o MS Auth eleito na T1 para autenticação.

Cada grupo é responsável por um dos domínios do sistema (Produto, Estoque, Pedidos, etc.), devendo entregar:

- API REST documentada com Swagger
- Microfrontend React + TypeScript publicado no NPM
- Pipeline CI/CD no GitHub Actions
- Integração com o MS Auth via JWT
