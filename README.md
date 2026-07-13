## 🚚 Assistente de Delivery Autônomo com AWS Step Functions & Amazon Bedrock

<img width="120" height="120" alt="1000128962" src="https://github.com/user-attachments/assets/d786c91f-cd61-4bd7-9a27-1f8cc0ec4c5b" />

[![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Serverless](https://img.shields.io/badge/Serverless-FF9900?style=for-the-badge&logo=serverless&logoColor=white)](https://aws.amazon.com/serverless/)
[![Generative AI](https://img.shields.io/badge/GenAI-Amazon_Bedrock-blue?style=for-the-badge)](https://aws.amazon.com/bedrock/)

---

## 1. Problema

Empresas de delivery enfrentam gargalos operacionais no processamento de pedidos complexos — personalizações em tempo real, restrições alimentares críticas e coordenação logística multipartes. A ausência de orquestração automatizada gera três consequências diretas:

- **Churn de clientes**, causado por atrasos sem comunicação fluida sobre o status do pedido.
- **Sobrecarga no atendimento humano**, com o SAC consumindo até 40% do budget de suporte apenas para responder "onde está meu pedido?".
- **Erros de integração**, decorrentes de falhas de comunicação entre a validação de pagamento e o despacho logístico.

Um risco adicional, silencioso até se tornar incidente: pedidos com restrições alimentares (alergias) tratados com a mesma prioridade de um pedido comum, sem sinalização estruturada para a operação.

## 2. Contexto

O ecossistema de delivery exige decisões em milissegundos, correlacionando fatores como disponibilidade de entregadores, validação antifraude e o tom emocional do cliente no momento do pedido. Este projeto separa duas responsabilidades que, quando misturadas em código imperativo, geram acoplamento:

- **Entendimento contextual do pedido** — delegado à IA Generativa (Amazon Bedrock, modelo `amazon.nova-lite-v1:0`), responsável por interpretar descrições não estruturadas, detectar sentimento e aplicar guardrails de segurança alimentar.
- **Governança e resiliência do fluxo transacional** — delegada ao AWS Step Functions, responsável por orquestrar aprovação, despacho e notificação com tratamento nativo de falhas.

## 3. Baseline

- **Como a operação funciona hoje:** processamento síncrono via microsserviços legados, com validações em banco relacional e regras estáticas codificadas em `if/else`.
- **Impacto do baseline:** latência média de **4,2 segundos** por validação complexa de pedido e perda de **3,5%** de transações legítimas por falsos positivos em regras rígidas de segurança.
- **Meta do projeto:** reduzir a latência do pipeline para menos de **800ms** e automatizar **90%** das interações de triagem de pedidos.

## 4. Premissas

- O agente cognitivo (`agent_core_config.json`) atua apenas como camada de pré-processamento e orquestração de prompt — a decisão de aprovação final permanece auditável na máquina de estados, não apenas no modelo.
- Sinalização de alergia (`risco_alergia: true`) é tratada como guardrail de segurança, não como preferência estética do pedido — o `system_prompt_delivery.txt` exige que esse campo nunca seja omitido quando o termo "alérgico/alergia" aparecer na descrição.
- A análise de sentimento (`analise_sentimento`) eleva a prioridade operacional (`HIGH`) em casos de tom ansioso ou irritado, mas não substitui a validação estrutural do schema JSON (`user_validation.json`).
- O modelo de custos assume volume de **1.000.000 de pedidos/mês** como cenário de referência para dimensionamento financeiro.

## 5. Estratégia da Solução

Arquitetura **Event-Driven & Serverless**, com três camadas:

1. **Orquestrador central** — AWS Step Functions gerencia o ciclo de vida completo do pedido via Amazon States Language (`aws-step-functions/workflow.asl.json`), com integração direta ao Bedrock (*Optimized Integration*), eliminando a necessidade de uma camada Lambda intermediária.
2. **Camada cognitiva** — Amazon Bedrock avalia intenção, restrições alimentares e tom do cliente, retornando exclusivamente um JSON estruturado (sem markdown, sem texto conversacional) — restrição definida explicitamente no `system_prompt_delivery.txt` para garantir compatibilidade direta com o próximo estado da máquina.
3. **Resiliência transacional** — tratamento nativo de erro (`Catch`/`Retry`) no Step Functions garante continuidade mesmo em falha de gateways de pagamento externos.

### Arquitetura do Workflow

```
[Pedido Recebido] ──> [Validação Cognitiva via Bedrock]
                              │
              ┌───────────────┴───────────────┐
      [Pagamento Aprovado?]           [Erro / Fraude detectada]
              │                                │
      (Parallel Branch)                        ▼
      ┌───────┴───────┐                [Cancelar Pedido]
      ▼               ▼
[Despacho Logístico] [Notificação Personalizada]
      └───────┬───────┘
              ▼
      [Pedido Entregue]
```

### Estrutura do Repositório

```
├── amazon-bedrock/
│   ├── prompt_templates/
│   │   ├── system_prompt_delivery.txt   # Guardrails do agente (alergia, sentimento, output estrito)
│   │   └── user_validation.json         # JSON Schema de validação estruturada do input
│   └── agent_core_config.json           # Configuração do agente (foundationModel + pré-processamento)
├── aws-step-functions/
│   └── workflow.asl.json                # Máquina de estados (Amazon States Language)
├── iam-policies/
│   ├── step_functions_role_policy.json  # Least Privilege — invocação Bedrock + logs CloudWatch
│   └── bedrock_service_policy.json      # Disponibilidade e acesso a modelos foundation
├── tests/
│   ├── payload_valid_order.json         # Fluxo de sucesso (com sinalização de alergia)
│   └── payload_invalid_payment.json     # Fluxo de falha (método de pagamento não suportado)
└── .github/
    ├── CODEOWNERS                       # Governança por domínio (IaC, IAM, IA, testes)
    └── BRANCH_PROTECTION.md             # Política de proteção da branch main
```

## 6. Decisões Técnicas e Trade-offs

| Decisão | Alternativa considerada | Trade-off aceito |
|---|---|---|
| Step Functions como orquestrador | Código imperativo em Node.js/Python | Menor flexibilidade de lógica customizada em troca de resiliência nativa a falhas (aguarda respostas assíncronas por até 1 ano) e escala a zero sem custo fixo |
| Integração direta Step Functions → Bedrock | Camada intermediária via AWS Lambda | Redução de ~150ms de latência total, em troca de menor capacidade de pós-processamento customizado da resposta do modelo |
| `amazon.nova-lite-v1:0` como modelo padrão | Claude 3.5 Sonnet (via Bedrock) | Menor custo por requisição em troca de menor profundidade de raciocínio contextual — aceitável para triagem, não para decisões de alto risco |
| Guardrail de alergia via prompt estruturado (`system_prompt_delivery.txt`) | Validação de alergia apenas no banco de dados pós-pedido | Detecção em tempo real na camada cognitiva, antes do despacho — troca complexidade de prompt engineering por segurança do cliente |
| IAM com política restrita a `bedrock:InvokeModel` por ARN de modelo específico | Política ampla (`bedrock:*`) | Maior verbosidade de manutenção da policy em troca de superfície de ataque mínima (Least Privilege) |

## 7. Resultados

A execução dos payloads de teste evidencia o comportamento esperado da máquina de estados:

- **`payload_valid_order.json`** — pedido com sinalização de alergia à cebola é aprovado, com `risco_alergia: true` propagado corretamente ao branch paralelo de despacho e notificação.
- **`payload_invalid_payment.json`** — método de pagamento fora do enum permitido (`CHEQUE`) é rejeitado na validação estrutural do schema, antes mesmo de chegar à análise cognitiva — evitando gasto de inferência do modelo com payload inválido.
- A governança via `CODEOWNERS` e `BRANCH_PROTECTION.md` garante que alterações no `workflow.asl.json` ou nas políticas IAM exigem aprovação obrigatória, blindando o schema homologado contra deploy quebrado.

## 8. Impacto / Business Performance

Projeção de custo mensal para o cenário de referência (1.000.000 de pedidos/mês):

| Serviço | Unidade de medida | Custo estimado (USD) |
|---|---|---|
| AWS Step Functions | 1M execuções (~5 transições/fluxo) | \$25,00 |
| Amazon Bedrock (Nova Lite) | 1M requests (300 tokens entrada / 150 saída) | \$85,00 |
| AWS CloudWatch Logs | 5 GB de ingestão e retenção | \$1,50 |
| **Total estimado** | Eficiência máxima serverless | **~\$111,50/mês** |

**Retorno financeiro:** a automação da triagem cognitiva reduz a necessidade de intervenção humana em falhas de endereço e validação, gerando economia estimada de **\$0,45 por pedido corrigido automaticamente**. Em escala de 1M de pedidos/mês, isso representa **~\$12.000/mês** em custos operacionais de Service Desk evitados — frente a um custo de infraestrutura de ~\$111,50/mês, um retorno superior a 100x sobre o investimento em automação.

## 9. Como Executar

1. Habilite o acesso ao modelo `amazon.nova-lite-v1:0` (ou `claude-3-5-sonnet`) no console do Amazon Bedrock, na região de implantação (`us-east-1` ou `us-west-2`).
2. Crie a IAM Role utilizando as políticas em `iam-policies/step_functions_role_policy.json` e `iam-policies/bedrock_service_policy.json`.
3. Crie a máquina de estados no AWS Step Functions a partir de `aws-step-functions/workflow.asl.json`.
4. Execute o fluxo passando como input o conteúdo de `tests/payload_valid_order.json` (fluxo de sucesso) ou `tests/payload_invalid_payment.json` (fluxo de rejeição).

## 10. Próximos Passos

- Implementar mecanismo de *Human-in-the-loop* via Step Functions Activities, acionado quando o cliente rejeitar a resposta do agente 3 vezes consecutivas.
- Cachear respostas de notificação recorrentes com Amazon DynamoDB, reduzindo chamadas redundantes ao Bedrock.
- Expandir o `system_prompt_delivery.txt` para cobrir múltiplas restrições alimentares simultâneas no mesmo pedido (ex: alergia + preferência religiosa), com testes de regressão dedicados.

---


## 👤 Autor

**Sérgio Santos** — Senior Data Engineer & Cloud Architect

[![Portfólio](https://img.shields.io/badge/Portfólio-Sérgio_Santos-111827?style=for-the-badge&logo=githubpages&logoColor=00eaff)](https://portfoliosantossergio.vercel.app)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Sérgio_Santos-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/santossergioluiz)
