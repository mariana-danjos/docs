# Critérios Avaliativos

## Avaliação do Professor (70%)

| Critério                | Peso | Insuficiente (0–49%) | Regular (50–69%) | Bom (70–89%) | Excelente (90–100%) |
|------------------------|------|----------------------|------------------|--------------|----------------------|
| **Governança e ADR**   | 20%  | ADR ausente ou sem justificativas técnicas mínimas. | ADR básico, descrevendo tecnologias mas sem análise de trade-offs. | ADR bem estruturado, justificando a stack. | ADR exemplar; justifica decisões de design para o domínio. |
| **MS de Autenticação** | 25%  | Não funcional ou sem padrões de segurança mínimos. | Funcional, mas com falhas no fluxo de JWT ou sem Refresh Token. | Implementação sólida com documentação Swagger completa. | Robusto; RBAC funcional, conteinerizado com Docker e integrado ao Ministack. |
| **Microfrontend (MFE)**| 20%  | Não utiliza a stack obrigatória. | Utiliza React/MUI, mas com inconsistências. | MFE funcional, seguindo o design system (MUI). | Interface fluida; manual de UI claro e tipagem rigorosa com TypeScript. |
| **DevOps e Qualidade** | 20%  | Sem pipelines de CI/CD ou sem testes unitários. | Pipeline existe, mas falha ou não gera releases no Docker Hub/NPM. | CI/CD funcional no GitHub Actions com execução de testes unitários. | Pipeline otimizado; alta cobertura de testes e geração automática de releases. |
| **Uso de IA: Logs e Crítica** | 15% | Sem registro de logs ou uso de código com "alucinações" óbvias e erros de sintaxe. | Logs básicos ou parciais; demonstrou pouco discernimento sobre o código gerado pela IA. | Logs organizados e evidência de revisão humana para ajustar o código à arquitetura. | Logs detalhados com evidências de uso crítico e reforço das decisões de design. |

---

## Avaliação dos Pares (15%)

| Eixo de Avaliação        | Critério de Observação                                                                 | Peso |
|--------------------------|----------------------------------------------------------------------------------------|------|
| **Clareza Arquitetural** | O grupo explicou bem como as decisões de design?                                      | 30%  |
| **Qualidade da Demo**    | A funcionalidade de autenticação e a interface pareceram estáveis?                    | 30%  |
| **Viabilidade de Adoção**| O serviço de Auth apresentado parece robusto o suficiente para ser o padrão da turma? | 25%  |
| **Documentação**         | O manual para UI e a documentação Swagger facilitam o entendimento por terceiros?     | 15%  |

---

## Autoavaliação (15%)

| Eixo de Avaliação        | Critério de Reflexão                                                                 | Peso |
|--------------------------|--------------------------------------------------------------------------------------|------|
| **Domínio Tecnológico**  | Evoluímos no uso de TypeScript, Docker, Ministack e GitHub Actions?                 | 30%  |
| **Uso Crítico de IA**    | Como utilizamos a IA para resolver problemas técnicos e o quanto revisamos o que ela gerou? | 25%  |
| **Gestão e Processo**    | Mantivemos o Kanban e os repositórios organizados conforme as regras da disciplina? | 25%  |
| **Resolução de Desafios**| Qual foi nossa capacidade de superar impedimentos técnicos (ex: ambiente, bugs)?    | 20%  |