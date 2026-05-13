# Sobre o Trabalho

Ao longo da disciplina, iremos desenvolver um **Sistema de Gestão de Estoque para Loja de Roupas Plus Size**, em parceria com Josiel Pereira.

O sistema é construído em arquitetura de **microsserviços com microfrontends**, onde cada grupo é responsável por um par MS + MFE de um domínio específico. Os serviços se comunicam via **AWS API Gateway** e **SQS/SNS**, e são deployados no **Amazon ECS (Fargate)**.

O projeto é dividido em duas entregas:

- **T1 — MS Auth + MFE de Autenticação:** cada grupo implementa o serviço de autenticação; a turma vota o melhor, que é adotado por todos na T2.
- **T2 — MS de Domínio + MFE:** cada grupo desenvolve seu microsserviço de domínio (Produto, Estoque, Pedidos etc.) integrado ao MS Auth eleito.

O trabalho aplica na prática conceitos de arquitetura distribuída, CI/CD, Module Federation, deploy em cloud e desenvolvimento colaborativo em equipes independentes.
