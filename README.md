## Criando um Assistente de Delivery com AWS Step Functions e Bedrock.

<img width="120" height="120" alt="1000128962" src="https://github.com/user-attachments/assets/d786c91f-cd61-4bd7-9a27-1f8cc0ec4c5b" />


Bootcamp - AWS - Agentes de IA em Campo


---

```

├── README.md                          # Documentação executiva e técnica (Narrativa Framework CDS)
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

