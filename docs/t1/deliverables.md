# Entregáveis

## Visão Geral

Na primeira entrega (T1), os grupos deverão apresentar uma solução inicial focada na infraestrutura base do sistema, contemplando autenticação, arquitetura, frontend e práticas de engenharia de software.

---

## Entregáveis Obrigatórios

### Autenticação e Autorização
- Implementação de um serviço de autenticação
- Controle de acesso (ex: JWT, RBAC, etc.)

---

### Documento de Arquitetura (ADR)
- Registro das decisões arquiteturais
- Justificativa das tecnologias escolhidas
- Análise de trade-offs

---

### Microsserviço
- Implementação de um microsserviço funcional
- Estruturado conforme boas práticas
- Executável em ambiente local

---

### Microfrontend (MFE)
- Interface desenvolvida com:
    - React + TypeScript (obrigatório)
    - MUI como design system (obrigatório)
- Integração com o microsserviço

---

### CI/CD (Integração e Entrega Contínua)
- Pipeline configurado (GitHub Actions)
- Execução automática de:
  - Testes unitários
  - Build da aplicação
- Geração de releases

---

### Documentação
- **Swagger/OpenAPI** para o microsserviço
- **Manual de UI** para o microfrontend

---

### IA
- Logs de uso da IA com a extensão [Congnitrace](https://marketplace.visualstudio.com/items?itemName=schardosim.cognitrace) do VSCode, ou os hooks disponíveis [neste repositório](https://github.com/simaoj/prompt-auto-log)