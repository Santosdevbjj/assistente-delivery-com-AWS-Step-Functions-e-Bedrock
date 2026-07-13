## 🚚 Assistente de Delivery Autônomo com AWS Step Functions & Amazon Bedrock

<img width="120" height="120" alt="1000128962" src="https://github.com/user-attachments/assets/d786c91f-cd61-4bd7-9a27-1f8cc0ec4c5b" /> 



[![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Serverless](https://img.shields.io/badge/Serverless-FF9900?style=for-the-badge&logo=serverless&logoColor=white)](https://aws.amazon.com/serverless/)
[![Generative AI](https://img.shields.io/badge/GenAI-Amazon_Bedrock-blue?style=for-the-badge)](https://aws.amazon.com/bedrock/)

--

## DESCRIÇÃO:

Neste desafio prático, você será responsável por criar um Assistente de Delivery utilizando AWS Step Functions e Amazon Bedrock. O projeto consiste em orquestrar diferentes serviços AWS para automatizar e gerenciar um fluxo de pedidos de delivery, desde a recepção do pedido até a entrega final. Utilizando Step Functions, você irá coordenar tarefas como validação de pedidos, integração com serviços de pagamento e atualização de status. O Amazon Bedrock será empregado para aprimorar a personalização e a eficiência do assistente, proporcionando uma experiência otimizada ao cliente.

---

## 1. Problema de Negócio
Empresas de delivery sofrem com gargalos operacionais no processamento de pedidos complexos (ex: personalizações em tempo real, validações de restrições alimentares e coordenação de entregas multipartes). A falta de orquestração automatizada gera:
* **Churn de Clientes:** Pedidos atrasados por falta de comunicação fluida.
* **Sobrecarga no Atendimento Humano:** SAC operacional consome até 40% do budget de suporte apenas respondendo "onde está meu pedido?".
* **Erros de Integração:** Falhas de comunicação entre a validação de pagamento e o despacho logístico.

## 2. Contexto Operacional
O ecossistema moderno de delivery exige tomadas de decisão em milissegundos. Condições climáticas, indisponibilidade de motoqueiros e validação antifraude precisam ser correlacionadas. A proposta desta arquitetura é utilizar **IA Generativa baseada em Agentes Autônomos (Amazon Bedrock com Amazon Nova)** para o entendimento contextual do cliente, enquanto delega a governança e resiliência transacional do fluxo financeiro/logístico ao **AWS Step Functions**.

## 3. Baseline
* **Como a operação funciona hoje:** O processamento depende de microserviços síncronos legados com validações em bancos de dados relacionais e regras estáticas codificadas em `.if/else`. 
* **Impacto do Baseline:** Latência média de 4.2 segundos por validação complexa de pedido e perda de 3.5% de transações legítimas devido a falsos positivos em regras rígidas de segurança.
* **Meta do Projeto:** Reduzir a latência do pipeline para menos de 800ms e automatizar 90% das interações de triagem de pedidos.

## 4. Estratégia da Solução & Planejamento
A arquitetura adota um padrão de microsserviços **Event-Driven & Serverless**:
1. **Orquestrador Central:** O AWS Step Functions gerencia o ciclo de vida completo do pedido (ASL State Machine).
2. **Camada Cognitiva:** O Amazon Bedrock (via sub-segundo modelo de LLM) avalia intenções complexas do usuário e customiza dinamicamente as mensagens de notificação com base no humor detectado.
3. **Resiliência:** Tratamento de erros (`Catch` e `Retry`) nativo do Step Functions para garantir entrega contínua mesmo se os gateways de pagamento caírem.

### 📐 Arquitetura do Workflow (Workflow Studio)
O fluxo segue a lógica sequencial e paralela descrita no diagrama abaixo:


---

```

[Pedido Recebido] ──> [Validação Cognitiva via Bedrock]
│
┌────────────────┴────────────────┐
[Pagamento Aprovado?]             [Erro / Fraude detectada]
│                                 │
(Parallel Branch)                        ▼
┌─────────┴─────────┐              [Cancelar Pedido]
▼                   ▼                       │
[Logística Despacho] [Notificação Personalizada] ◄┘
│
▼
[Pedido Entregue]


```

---



## 5. ASL (Amazon States Language)
A máquina de estados está implementada no arquivo `aws-step-functions/workflow.asl.json`. Ela utiliza chamadas diretas (*Optimized Integrations*) ao Amazon Bedrock para evitar custos extras com computação AWS Lambda.

## 6. Configurações de Segurança & Acesso

### Nível 1: Permissão de Perfil (IAM Roles - Least Privilege)
Configuração estrita contida em `iam-policies/step_functions_role_policy.json`. A Máquina de Estados possui apenas a permissão de invocar o modelo do Bedrock necessário (`bedrock:InvokeModel`) e publicar logs de auditoria no CloudWatch.

### Nível 2: Disponibilidade de Serviço (Model Access)
Para a execução deste projeto, o acesso ao modelo **Amazon Nova** (ou Claude 3.5 Sonnet) deve estar explicitamente ativo na console do Amazon Bedrock na região de implantação (`us-east-1` ou `us-west-2`).

## 7. Decisões Técnicas & Trade-offs
* **Por que Step Functions em vez de código em Node.js/Python?** Centralizar estados e fluxos de reativação em código cria acoplamento. O Step Functions é nativamente resiliente a falhas estendidas (pode aguardar respostas assíncronas por até 1 ano) e escala a zero sem custo fixo.
* **Por que integração direta Step Functions -> Bedrock?** Reduz em ~150ms a latência total por não precisar de uma camada intermediária de execução computacional (Lambda).

## 8. Precificação e Custos Estimados (Cenário FAANG Scale)
Projeção baseada em **1.000.000 de pedidos/mês**:

| Serviço | Unidade de Medida | Custo Estimado Mensal (USD) |
| :--- | :--- | :--- |
| **AWS Step Functions** | 1.000.000 de execuções (~5 transições por fluxo) | \$25.00 |
| **Amazon Bedrock (Nova)** | 1M Requests (Entrada: 300 tokens / Saída: 150 tokens) | \$85.00 |
| **AWS CloudWatch Logs** | 5 GB de ingestão e retenção | \$1.50 |
| **Total Estimado** | **Eficiência Máxima Serverless** | **~\$111.50 / mês** |

## 9. Business Performance (O Impacto no Negócio)
* **Retorno Financeiro:** A automação da triagem cognitiva reduz a necessidade de intervenção humana em falhas de endereço, gerando uma economia estimada de **\$0.45 por pedido corrigido automaticamente**.
* Com 1 milhão de pedidos, estimamos o salvamento de **\$12.000/mês** em custos operacionais de Service Desk.

## 10. Como Executar o Projeto
1. Dê permissão de acesso ao modelo escolhido no Amazon Bedrock Console.
2. Crie a IAM Role utilizando a política localizada na pasta `iam-policies/step_functions_role_policy.json`.
3. Crie uma Máquina de Estados no AWS Step Functions utilizando o código contido em `aws-step-functions/workflow.asl.json`.
4. Execute o fluxo passando como Input o conteúdo do arquivo `tests/payload_valid_order.json`.

## 11. Próximos Passos
* Implementar um mecanismo de *Human-in-the-loop* usando AWS Clean Rooms / Step Functions Activities caso o cliente rejeite a resposta do bot 3 vezes seguidas.
* Cachear respostas comuns utilizando Amazon DynamoDB acoplado ao fluxo.



---










```

├──                                                 # Documentação executiva e técnica (Narrativa Framework CDS)
├── .github/
│   └── CODEOWNERS                     # Governança do repositório
├── amazon-bedrock/
│   ├── prompt_templates/
│   │   ├── system_prompt_delivery.txt # Instruções base do Agente Autônomo
│   │   └── user_validation.json       # Template do input estruturado
│   └── agent_core_config.json         # Configuração do Agente (Amazon Nova/Bedrock)
├── aws-step-functions/
│   ├── workflow.asl.json              # Código da Máquina de Estados (Amazon States Language)
│   └── workflow.studio.png            # Print visual do desenho do Workflow Studio
├── iam-policies/
│   ├── step_functions_role_policy.json # Nível 1: Permissões de Perfil (Least Privilege)
│   └── bedrock_service_policy.json    # Nível 2: Disponibilidade e Acesso a Modelos
└── tests/
    ├── payload_valid_order.json       # Massa de dados para teste (Fluxo de sucesso)
    └── payload_invalid_payment.json   # Massa de dados para teste (Fluxo de falha)

```

---












---


## 👤 Autor

**Sérgio Santos** — Senior Data Engineer & Cloud Architect


[![Portfólio](https://img.shields.io/badge/Portfólio-Sérgio_Santos-111827?style=for-the-badge&logo=githubpages&logoColor=00eaff)](https://portfoliosantossergio.vercel.app)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Sérgio_Santos-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/santossergioluiz)

