# Banco de Dados

![PostgreSQL](../../images/pg.png){ class="tech-logo" }
### PostgreSQL
[https://www.postgresql.org/](https://www.postgresql.org/){ class="tech-link" }

- **Quando usar:** quer banco relacional robusto com suporte a relações complexas e integridade forte.
- **Pontos fortes:** ACID completo, excelente suporte a índices e queries avançadas, JSONB nativo, amplamente utilizado no mercado.
- **Pontos de atenção:** exige modelagem relacional bem definida e migrations organizadas.

---

![MongoDB](../../images/mongodb.svg){ class="tech-logo" }
### MongoDB
[https://www.mongodb.com/](https://www.mongodb.com/){ class="tech-link" }

- **Quando usar:** domínio naturalmente orientado a documentos JSON (ex: evento com sessões embutidas).
- **Pontos fortes:** flexibilidade de esquema, rápida evolução do modelo, integração natural com APIs REST.
- **Pontos de atenção:** relações complexas devem ser cuidadosamente modeladas (embedding vs referencing).

---

![DynamoDB](../../images/dynamodb.png){ class="tech-logo" }
### DynamoDB
[https://aws.amazon.com/dynamodb/](https://aws.amazon.com/dynamodb/){ class="tech-link" }

- **Quando usar:** quer explorar banco NoSQL escalável e modelagem orientada a padrões de acesso.
- **Pontos fortes:** alta performance, serverless, integração nativa com AWS, escalabilidade automática.
- **Pontos de atenção:** modelagem é baseada em access patterns, não em relações; exige planejamento prévio das queries.
