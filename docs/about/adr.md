# ADR — Documento de Decisão de Arquitetura
### Plus - Sistema de Gestão de Estoque para Loja de Roupas Plus Size

| Campo | Valor |
|---|---|
| **Projeto** | Sistema de gestão de estoque para loja de roupas plus size |
| **Disciplina** | Engenharia de Software II — 98802-02 |
| **Turma** | 30 — PUCRS / 2026-1 |
| **Professor** | Prof. José Pedro Schardosim Simão |
| **Parceiro** | Josiel Pereira — Araranguá/SC |
| **Versão / Data** | v1.1 — Abril de 2026 |
| **Status** | ✔ Aceita |

---

## 1. Contexto e Problema

Sistema de Gestão de Estoque para loja de roupas plus size, com suporte a grade extendida (busto/cintura/quadril), variação de medidas por fabricante e consulta rápida para vendedores.

Este ADR documenta as decisões sobre microsserviços, microfrontends, autenticação, comunicação entre serviços, tecnologias e infraestrutura.

---

## 2. Decisão Arquitetural

Adotar uma arquitetura de microsserviços com microfrontend, composta por 10 unidades deployáveis independentes (1 serviço de Autenticação/Autorização + 9 serviços de domínio), cada uma com seu respectivo microfrontend React.

> **Decisão Central:** 10 microsserviços + 10 microfrontends React, orquestrados por um Shell App via AWS API Gateway. Comunicação assíncrona via SQS/SNS. Infra: ECS Fargate (MSs), RDS (banco), S3 + CloudFront (frontend), Ministack (dev local). Cada grupo é responsável por um par MS + MFE; o MS Auth será escolhido por votação após a P1.

---

## 3. Mapeamento dos Microsserviços e Microfrontends

| ID | Nome | Responsabilidade do MS | Responsabilidade do MFE |
|---|---|---|---|
| **MS Auth** | Autenticação / Autorização | Login, OAuth2/OIDC, emissão e refresh de JWT, RBAC por roles, revogação de tokens | Telas de login/logout, gerenciamento de sessão, contexto de auth compartilhado no Shell |
| **MS1** | Produto | Cadastro de produtos, tabelas de medidas (busto/cintura/quadril), manequim numérico, variações de cor e grade | Cadastro de peças, tabela de medidas, variações de cor/grade |
| **MS2** | Categorização | Gerenciamento de categorias, tags e coleções (moda praia, casual, festa etc.) | Árvore de categorias, tags e coleções, regras de exibição |
| **MS3** | Mídia | Upload e armazenamento de imagens por variação, galeria por cor/tamanho, ordem de exibição | Upload de fotos, galeria por variação, ordenação de imagens |
| **MS4** | Estoque / Grade | Entradas e saídas de estoque, saldo por grade de tamanhos, histórico de movimentações, ajuste manual | Saldo por grade, entrada de mercadoria, histórico de movimentos, ajuste manual |
| **MS5** | Alertas | Monitoramento de baixo estoque por grade, configuração de limiares por tamanho, notificações push | Lista de alertas ativos, configuração de limiares, notificações push |
| **MS6** | Consulta Rápida | API de busca de disponibilidade para vendedores, busca por peça, tamanho e localização física | Busca rápida por peça, disponibilidade por tamanho, localização física no estoque |
| **MS7** | Pedidos | Registro de pedidos/vendas, baixa automática no estoque, lista de status e reserva de grade | Novo pedido/venda, lista e status de pedidos, histórico de saídas, reserva de grade |
| **MS8** | Painel Operacional | Feed de eventos em tempo real de entradas/saídas de mercadorias | Entradas/saídas em tempo real, feed de eventos live |
| **MS9** | Relatórios / Analytics | Geração de relatórios de vendas por categoria e faixa de tamanho, desempenho por grade, exportação de dados | Relatórios por categoria, desempenho por grade, exportação de dados |

---

## 4. Visão da Arquitetura

### 4.1 Fluxo Geral

O browser carrega o **Shell App** (S3 + CloudFront), que gerencia roteamento, design system e contexto de auth. Cada MFE é carregado via Module Federation (lazy loading).

As requisições passam pelo **AWS API Gateway**, que valida o JWT por introspecção e roteia para o MS correspondente no **ECS Fargate**. Os MSs recebem os claims do JWT via header, sem revalidar o token.

Comunicação assíncrona entre MSs (ex: pedido → baixa de estoque) é feita via **SQS/SNS**. No desenvolvimento local, o **Ministack** emula todos os serviços AWS sem custo.

### 4.2 Camadas da Arquitetura

| Camada | Serviço AWS | Descrição |
|---|---|---|
| **Shell App** | S3 + CloudFront | Build React servido via CDN; gerencia rotas, design system (MUI), shared state (AuthContext) |
| **MFEs** | S3 + CloudFront | 9 microfrontends React + TypeScript, publicados como pacotes NPM, integrados via Module Federation |
| **API Gateway** | AWS API Gateway | Roteamento para microsserviços, rate limit, validação JWT por introspecção, contrato OpenAPI |
| **MS Auth** | ECS Fargate + RDS | Autenticação e autorização; emite e revoga JWTs; User Store em banco relacional (RDS) |
| **Microsserviços (8)** | ECS Fargate + RDS/S3 | Cada MS expõe uma API REST documentada com Swagger; recebe claims do JWT via header |
| **Event Bus** | Amazon SQS / SNS | Comunicação assíncrona entre microsserviços de domínio |
| **Armazenamento de arquivos** | Amazon S3 | Imagens de produtos (MS Mídia), exportações de relatórios (MS9) |
| **Banco de dados** | Amazon RDS (PostgreSQL) | Banco relacional por microsserviço (padrão database-per-service) |
| **Registro de imagens** | Amazon ECR | Repositório de imagens Docker dos microsserviços |
| **Dev local** | Ministack | Emulação local de todos os serviços AWS (S3, SQS, SNS, RDS, API GW, ECR) sem custo |

---

## 5. Serviços AWS Utilizados

| Serviço AWS | Papel na Arquitetura | Emulado no Ministack |
|---|---|---|
| **Amazon ECS (Fargate)** | Execução dos containers dos microsserviços sem gerenciar servidores | ✔ |
| **Amazon ECR** | Registro privado de imagens Docker dos microsserviços | ✔ |
| **AWS API Gateway** | Ponto de entrada único, rate limit, roteamento e validação JWT | ✔ |
| **Amazon RDS (PostgreSQL)** | Banco de dados relacional, um por microsserviço (database-per-service) | ✔ |
| **Amazon S3** | Hospedagem do Shell App e MFEs (estáticos), armazenamento de imagens e exports | ✔ |
| **Amazon CloudFront** | CDN para distribuição do frontend com baixa latência | — |
| **Amazon SQS** | Filas de mensagens para comunicação assíncrona entre MSs (ex: pedido → estoque) | ✔ |
| **Amazon SNS** | Publicação de eventos/notificações (ex: alerta de baixo estoque) | ✔ |
| **AWS IAM** | Controle de permissões entre serviços AWS (roles por MS) | ✔ |
| **AWS CloudWatch** | Logs centralizados e métricas dos microsserviços | ✔ |

> **Ministack** é um emulador leve dos serviços AWS (S3, SQS, SNS, RDS, API Gateway), compatível com o AWS SDK. Não requer conta AWS ativa durante o desenvolvimento.

---

## 6. Tecnologias Adotadas

### 6.1 Microfrontend

- **Obrigatório:** React + TypeScript
- **Design system:** MUI (Material UI)
- **Module Federation:** Webpack 5 ou Vite
- **CI/CD:** GitHub Actions
- **Distribuição:** pacotes publicados no NPM; hospedagem via S3 + CloudFront
- Testes unitários com cobertura mínima definida pela equipe

### 6.2 Microsserviços

- **Linguagens recomendadas:** TypeScript (Node.js), Python ou Ruby
- **Desenvolvimento local:** Docker + Ministack (emulação de serviços AWS)
- **CI/CD:** GitHub Actions (testes unitários + geração de release)
- **Release:** imagens Docker publicadas no Amazon ECR
- **Documentação:** Swagger (OpenAPI 3.0)

### 6.3 Infraestrutura e DevOps

- Repositórios: GitHub Org da disciplina, 1 repositório por MS/MFE
- CI/CD: GitHub Actions com pipeline de testes, build e push para ECR
- Ambiente local: Docker Compose + Ministack (emula S3, SQS, SNS, RDS, API Gateway, ECR)
- Deploy obrigatório: **AWS** (ECS Fargate + API Gateway + S3 + RDS + SQS/SNS)
- Testes de integração (ponto extra): suite separada da unitária

---

## 7. Microsserviço de Autenticação e Autorização (MS Auth)

### 7.1 Escopo

Único serviço responsável pela identidade dos usuários. Roda no **ECS Fargate** com User Store no **RDS (PostgreSQL)**. Os demais MSs confiam nos claims do JWT entregues pelo API Gateway via header, sem revalidar o token.

### 7.2 Responsabilidades

- Login com usuário/senha (e opcionalmente OAuth2/OIDC)
- Emissão de JWT (access token) e refresh token
- RBAC: controle de permissões por role (admin, vendedor, gestor)
- Revogação de tokens (logout, invalidação forçada)
- Persistência dos usuários em User Store (RDS PostgreSQL)
- Exposição de endpoint de introspecção para uso pelo AWS API Gateway

### 7.3 Processo de Seleção

Na P1 (13/05), cada grupo entrega seu MS Auth + MFE. Após a entrega, a turma vota o melhor par, que se torna a solução oficial para a entrega final (24/06).

---

## 8. Alternativas Consideradas

| Alternativa | Prós | Contras / Motivo da Rejeição |
|---|---|---|
| Monólito único | Simples de implementar inicialmente; sem overhead de rede entre módulos | Não atende ao requisito pedagógico da disciplina; não permite desenvolvimento paralelo por grupos; baixa escalabilidade |
| Monólito modular (sem deploy independente) | Mais organizado que monólito puro; sem latência de rede interna | Ainda não permite deploys independentes; conflitos de merge entre grupos; não cumpre requisito de microsserviços |
| MFE sem Module Federation (iframes) | Isolamento total de contexto | UX degradada; sem compartilhamento de estado; integrações complexas; não permite compartilhar design system |
| Serviço de auth externo (Auth0, Keycloak) | Pronto para produção; alta segurança | Não cumpre o requisito pedagógico de implementar o MS Auth; limita o aprendizado da turma |
| Comunicação síncrona pura (REST entre MSs) | Simplicidade de implementação | Cria acoplamento temporal; falhas em cascata; não escala bem para operações de baixa de estoque em pedidos |
| Localstack no lugar de Ministack | Mais conhecido e com documentação ampla | Footprint maior, mais difícil de configurar localmente; Ministack oferece setup mais simples e compatível com o AWS SDK |

---

## 9. Consequências

### 9.1 Positivas

- **Desenvolvimento paralelo:** cada grupo trabalha de forma independente no seu MS + MFE
- **Escalabilidade:** ECS Fargate escala cada serviço de forma independente sem gerenciar servidores
- **Resiliência:** falha em um serviço não derruba a plataforma inteira
- **Aprendizado pedagógico completo:** cada grupo domina um fluxo ponta-a-ponta com serviços AWS reais
- **Padronização:** AWS API Gateway + JWT + Swagger garante contrato claro entre equipes
- **Paridade local/produção:** Ministack garante que o ambiente de dev replica fielmente os serviços AWS de produção

### 9.2 Negativas / Riscos

- **Overhead de comunicação:** latência adicional nas chamadas entre serviços via API Gateway
- **Complexidade operacional:** orquestração de múltiplos serviços AWS (mesmo no Ministack)
- **Consistência eventual:** operações assíncronas via SQS/SNS exigem tratamento de falhas e idempotência
- **Curva de aprendizado:** AWS SDK, ECS, IAM roles e Module Federation podem ser novidade para parte da turma
- **Gestão de dependências:** o Shell App precisa compatibilidade com versões dos MFEs publicados no NPM
- **Custo AWS:** deploy real na AWS gera custos; grupos devem usar Free Tier e monitorar via CloudWatch

---

## 10. Histórico de Revisões

| Versão | Data | Autor | Descrição |
|---|---|---|---|
| v1.0 | Abr/2026 | Prof. | Versão inicial do ADR com mapeamento completo dos 9 MSs + MFEs, tecnologias, critérios de avaliação e cronograma |
| v1.1 | Abr/2026 | Prof. | Migração de Localstack para Ministack; adoção formal dos serviços AWS (ECS Fargate, API Gateway, S3, RDS, SQS/SNS, ECR, CloudWatch); deploy AWS promovido de ponto extra para parte da entrega final; seção de serviços AWS adicionada |

---

*Decisões arquiteturais relevantes que desviem deste documento devem ser registradas como nova versão do ADR, com justificativa, data e autores.*
