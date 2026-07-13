## Criando um Assistente de Delivery com AWS Step Functions e Bedrock.

<img width="120" height="120" alt="1000128962" src="https://github.com/user-attachments/assets/d786c91f-cd61-4bd7-9a27-1f8cc0ec4c5b" />


Bootcamp - AWS - Agentes de IA em Campo


---
DESCRIÇÃO:

Neste desafio prático, você será responsável por criar um Assistente de Delivery utilizando AWS Step Functions e Amazon Bedrock. O projeto consiste em orquestrar diferentes serviços AWS para automatizar e gerenciar um fluxo de pedidos de delivery, desde a recepção do pedido até a entrega final. Utilizando Step Functions, você irá coordenar tarefas como validação de pedidos, integração com serviços de pagamento e atualização de status. O Amazon Bedrock será empregado para aprimorar a personalização e a eficiência do assistente, proporcionando uma experiência otimizada ao cliente.

---

Assistente de Delivery com AWS Step Functions e Amazon Bedrock

Projeto desenvolvido para demonstrar a construção de um assistente inteligente de delivery utilizando AWS Step Functions para orquestração de processos e Amazon Bedrock para personalização com IA Generativa.

Visão Geral

O sistema automatiza o fluxo de pedidos de delivery, desde a recepção do pedido até a entrega final, garantindo validação, processamento de pagamento, atualização de status e comunicação inteligente com o cliente.

Problema de Negócio

Empresas de delivery precisam processar pedidos de forma rápida, segura e escalável. Processos manuais podem gerar atrasos, falhas de comunicação, erros de pagamento e baixa satisfação do cliente.

Objetivo do Projeto

Criar uma arquitetura serverless na AWS que:

- Receba e valide pedidos de delivery.
- Orquestre o fluxo completo do pedido com AWS Step Functions.
- Utilize Amazon Bedrock para gerar respostas personalizadas ao cliente.
- Atualize o status do pedido em cada etapa do processo.
- Demonstre boas práticas de segurança, escalabilidade e observabilidade.

Arquitetura da Solução

O fluxo principal é composto por:

1. Recepção do pedido via API ou evento.
2. Validação do pedido por função AWS Lambda.
3. Processamento de pagamento.
4. Atualização do status do pedido.
5. Uso do Amazon Bedrock para gerar mensagens personalizadas.
6. Notificação do cliente sobre o andamento do pedido.

Tecnologias Utilizadas

- AWS Step Functions
- AWS Lambda
- Amazon Bedrock
- Amazon API Gateway
- Amazon DynamoDB
- Amazon SNS
- AWS IAM
- AWS CloudWatch
- AWS CloudFormation
- Python 3.12

Estrutura do Repositório

Veja a organização completa das pastas na seção de estrutura do repositório.

Como Executar

Pré-requisitos

- Conta AWS ativa
- AWS CLI configurado
- Permissões para Step Functions, Lambda, Bedrock, DynamoDB e SNS
- Python 3.12

Deploy da infraestrutura

cd infrastructure/cloudformation
aws cloudformation deploy \
  --template-file stack.yaml \
  --stack-name assistente-delivery \
  --capabilities CAPABILITY_NAMED_IAM

Execução do workflow

aws stepfunctions start-execution \
  --state-machine-arn ARN_DA_STATE_MACHINE \
  --input file://step-functions/exemplos/pedido-valido.json

Decisões Técnicas

- AWS Step Functions foi escolhido pela capacidade de orquestrar fluxos complexos de forma visual e resiliente.
- Amazon Bedrock foi utilizado para adicionar IA Generativa ao assistente, permitindo respostas mais naturais e contextualizadas.
- AWS Lambda foi adotado para executar lógica de negócio de forma serverless e escalável.
- DynamoDB pode ser utilizado para armazenar pedidos e status de execução com baixa latência.

Aprendizados

- Orquestração de fluxos serverless com Step Functions.
- Integração de IA Generativa com Amazon Bedrock.
- Implementação de políticas IAM seguras para serviços AWS.
- Estruturação de projetos cloud com foco em legibilidade e manutenção.

Próximos Passos

- Implementar integração real com provedores de pagamento.
- Adicionar rastreamento em tempo real da entrega.
- Criar dashboard de monitoramento com Amazon CloudWatch.
- Implementar testes automatizados de integração.

Licença

Este projeto está licenciado sob a licença MIT.



---









---


## 👤 Autor

**Sérgio Santos** — Senior Data Engineer & Cloud Architect


[![Portfólio](https://img.shields.io/badge/Portfólio-Sérgio_Santos-111827?style=for-the-badge&logo=githubpages&logoColor=00eaff)](https://portfoliosantossergio.vercel.app)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Sérgio_Santos-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/santossergioluiz)



---
