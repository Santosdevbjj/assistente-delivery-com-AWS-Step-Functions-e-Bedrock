# 🛡️ Política de Proteção de Branch e Governança de Código (CI/CD)

Este documento descreve as regras de proteção aplicadas à branch `main` deste repositório, garantindo conformidade com padrões de engenharia de nível **FAANG**.

## 🚀 Regras Ativas na Interface do GitHub

Para mitigar erros em produção, alterações na infraestrutura de IA e orquestração de nuvem foram blindadas com as seguintes regras de proteção de branch:

1. **Revisão Obrigatória via Pull Request:** Nenhuma alteração pode ser dropada diretamente na branch `main`. Todo código deve passar por um ambiente de Pull Request de forma assíncrona.
2. **Aprovação Vinculada ao CODEOWNERS:** O gatilho de merge é estritamente bloqueado até que o proprietário definido no arquivo `.github/CODEOWNERS` realize a revisão e dê o *Approve*.
3. **Bloqueio de Commits Diretos de Administradores:** As regras aplicam-se a todos os membros do time, incluindo administradores, evitando bypass acidental de políticas de segurança.

## 🎯 Impacto no Negócio & Engenharia

* **Maturidade de Processo (Luiz Café Style):** Evita que códigos de teste alterem prompts de produção ou tabelas de precificação sem validação técnica de pares.
* **Estabilidade do Pipeline (Meigarom Lopes Style):** Garante que o schema de validação (`user_validation.json`) e a máquina de estados em ASL fiquem idênticos ao baseline homologado, blindando as métricas financeiras do negócio contra deploy quebrado.
