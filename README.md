# 🚨 Zabbix-Telegram Workflow Interativo com n8n 🤖

## Visão Geral
Este projeto contém o workflow n8n projetado para transformar alertas padrão do **Zabbix** em mensagens interativas e ricas em HTML no **Telegram**.

A solução permite que os usuários executem ações críticas **direto dos botões da mensagem**, como:
* ✅ **Reconhecer Problema** (chamando a API do Zabbix).
* 📝 **Ver Histórico** de ocorrências da trigger.
* 🔄 **Pedir Atualização** (Status ICMP do Host).
* 📊 Gera um **Resumo Diário** dos incidentes.

---

## Estrutura do Repositório

| Diretório/Arquivo | Conteúdo |
| :--- | :--- |
| `workflows/zabbix-telegram.json` | O código do workflow n8n (com placeholders de variáveis de ambiente). |
| `docs/code-examples.md` | Explicação detalhada dos blocos de código (Code Nodes) utilizados no fluxo. |
| `.env.example` | Template com todas as variáveis de ambiente necessárias (Tokens, URLs, etc.). |
| `README.md` | Você está aqui! Documentação principal e instruções. |

---

## ⚙️ Pré-requisitos e Configuração

Para que este workflow funcione, você precisa ter configurado:

1.  **Instância n8n:** Ativa e com acesso externo.
2.  **Telegram Bot:** Token de acesso (via @BotFather).
3.  **Zabbix:** Acesso à API para configuração do Webhook e autenticação via n8n.

### 1. Importação do Workflow

1.  Copie o conteúdo do arquivo `workflows/zabbix-telegram.json`.
2.  Altere as Variáveis de Ambiente usando o `.env.example` como base e preencha as suas chaves e URLs reais.
3.  No seu n8n, clique em **Add Workflow** (Adicionar Workflow) e escolha **Import from JSON** (Importar de JSON).

### ⚠️ Requisito Crítico de Compatibilidade do n8n

**ATENÇÃO:** Este workflow utiliza o recurso **Data Tables** dentro dos nós "Code" para estruturar e retornar dados complexos de forma otimizada.

| Recurso | n8n Versão Mínima |
| :--- | :--- |
| **Data Tables** (Renderização estruturada nos Code Nodes) | **n8n @1.113 ou superior** |

**Se você estiver em uma versão anterior à n8n @1.113, o workflow NÃO funcionará corretamente** nos nós que dependem dessa funcionalidade. É **obrigatório** atualizar sua instância do n8n para garantir a execução do fluxo.
