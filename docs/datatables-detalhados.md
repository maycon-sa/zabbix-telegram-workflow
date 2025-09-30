# 📋 Esquema das Tabelas de Dados (Data Tables)

Este workflow utiliza **Tabelas de Dados** para a persistência de informações cruciais, como tokens de API e o status das mensagens no Telegram (necessário para a edição e atualização dos alertas).

Se o seu n8n não usa um banco de dados externo, essas tabelas serão manipuladas pelo nó **DataStore**. A criação correta dessas estruturas é essencial para que o fluxo funcione.

---

## 1. Tabela: `Zabbix_Triggers`

Esta tabela armazena o status e os metadados de cada alerta, sendo essencial para a edição da mensagem (Resolução, Reconhecimento) e para a consulta de histórico.

| Campo (Coluna) | Tipo de Dado | Finalidade |
| :--- | :--- | :--- |
| **`event_id`** | String | **Chave Primária.** ID único do evento Zabbix. Usado para o Reconhecimento. |
| **`event_name`** | String | Nome completo da Trigger (descrição do problema). |
| **`host_name`** | String | Nome do Host afetado pelo problema. |
| **`host_ip`** | String | Endereço IP do Host. |
| **`item_name`** | String | Nome do Item que gerou o alerta. |
| **`item_value`** | String | Último valor coletado do Item no momento do alerta. |
| **`severety`** | String | Nível de severidade do problema (ex: *High*, *Disaster*). |
| **`status`** | String | Estado do alerta (sempre será **PROBLEM** ao ser criado). |
| **`event_date`** | String | Data de início do problema. |
| **`event_time`** | String | Hora de início do problema. |
| **`recovery_date`** | String | Data em que o problema foi resolvido. |
| **`recovery_time`** | String | Hora em que o problema foi resolvido. |
| **`trigger_id`** | String | ID da Trigger (usado na consulta de Histórico). |
| **`message_id`** | String | **ID da Mensagem no Telegram.** CRÍTICO para a edição da mensagem. |
| **`chat_id`** | String | ID do chat onde a mensagem original foi enviada. |
| **`resolvido`** | String | Flag customizada para indicar se o problema já tem uma resolução (ex: `0` para aberto, `1` para resolvido). |

---

## 2. Tabela: `Zabbix_Tokens`

Usada para armazenar o token de autenticação da API do Zabbix, evitando que o fluxo precise fazer login em toda e qualquer requisição de consulta. Este token possui um tempo de vida limitado (timeout da API Zabbix).

| Campo (Coluna) | Tipo de Dado | Finalidade |
| :--- | :--- | :--- |
| **`KEY`** | String | **Chave Primária.** Identificador da entrada do token. O valor (ex: **"CONSULTA / ESCRITA"**) deve ser consistente em todos os nós que buscam o token. |
| **`VALUE`** | String | O **Token de Sessão** (Auth ID) gerado pela API do Zabbix após a autenticação (`user.login`). Este valor é atualizado periodicamente ou fixo dependendo da sua escolha no zabbix. |

---

## 3. Tabela: `Telegram`

Usada para armazenar informações de usuários que interagem com o bot, permitindo respostas diretas e futuras personalizações de acesso.

| Campo (Coluna) | Tipo de Dado | Finalidade |
| :--- | :--- | :--- |
| **`CHAT_ID`** | String | **Chave Primária.** ID do usuário no Telegram (usado para resposta privada). |
| **`NOME`** | String | Nome completo do usuário. |
| **`USERNAME`** | String | @username do Telegram (se disponível). |
| **`FIRST_NAME`** | String | Primeiro nome do usuário. |
| **`STATUS`** | String | Status interno de registro do usuário, usado para outros fins e reaproveitado o bd no projeto (ex: `autorizado`,`nao_autorizado`). |
| **`GRUPO_USUARIO`**| String | Informa se é grupo ou usuario (ex: `GRUPO`, `USUARIO`). |
