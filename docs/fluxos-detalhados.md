# 🗺️ Mapa de Fluxos: Detalhamento Nó por Nó

Este documento detalha o que cada nó faz em cada um dos seis fluxos lógicos do workflow.

---

## Fluxo 1: Alerta de Problema (Zabbix para Telegram)

**Gatilho:** Recebimento de um alerta de PROBLEMA do Zabbix.

| Nó | Tipo | Função |
| :--- | :--- | :--- |
| **Webhook (Zabbix)** | Trigger | Ponto de entrada. Escuta na URL de produção a requisição POST enviada pelo Zabbix. |
| **Set (Extrair Dados)** | Transform | Pega os campos JSON brutos (`eventID`, `triggerID`, `hostName`, `status`, etc.) e os armazena como variáveis limpas. |
| **Switch (Problema ou Resolução)** | Logic | Filtra o status do alerta. Se for **"PROBLEM"**, segue para este fluxo. |
| **Code (Formatar Mensagem)** | Code | Cria o corpo da mensagem em HTML e anexa os botões interativos (Reconhecer, Histórico, Atualização). |
| **HTTP Request (Envia Problema)** | Communication | Envia a mensagem formatada ao Telegram (usando o `TELEGRAM_BOT_TOKEN` e `TELEGRAM_CHAT_ID`). |
| **Set (Salvar Message ID)** | Transform | Salva o `message_id` retornado pelo Telegram. Este ID é crucial para editar a mensagem quando a resolução ou o reconhecimento chega. |

---

## Fluxo 2: Resolução de Problema (Zabbix para Telegram)

**Gatilho:** Recebimento de um alerta de RESOLUÇÃO do Zabbix.

| Nó | Tipo | Função |
| :--- | :--- | :--- |
| **Switch (Problema ou Resolução)** | Logic | Filtra o status do alerta. Se for **"RESOLVED"**, segue para este fluxo. |
| **Code (Formata Mensagem Resolução Editar)** | Code | Monta o texto de resolução. Ele é projetado para **editar** a mensagem original, se ela for encontrada. |
| **Telegram (Editar Mensagem)** | Communication | Tenta editar a mensagem original (usando o `message_id` salvo). Se bem-sucedido, sinaliza a resolução e remove os botões. |
| **NoOp (Resolução com Erro)** | Utility | Nó de Continuação para o plano B em caso de erro na edição da mensagem. |
| **Code (Formata Mensagem Resolução)1** | Code | **Plano B:** Monta uma *nova* mensagem de resolução, caso a edição da mensagem original tenha falhado (ou o alerta inicial nunca tenha sido enviado). |
| **Telegram (Enviar Nova Mensagem)** | Communication | Envia a nova mensagem de resolução para o chat. |

---

## Fluxo 3: Ação "Reconhecer Problema"

**Gatilho:** Clique no botão `✅ Reconhecer Problema` no Telegram.

| Nó | Tipo | Função |
| :--- | :--- | :--- |
| **Telegram Trigger** | Trigger | Recebe e processa o *callback_data* (o clique no botão). |
| **Switch (Ação do Botão)** | Logic | Filtra o prefixo do *callback_data* (ex: `ack:`). |
| **Code (Extrair Callback_data)** | Code | Analisa o *callback_data* para extrair o `eventID` do Zabbix e o nome completo do usuário que clicou. |
| **HTTP Request (Zabbix API Login)** | Communication | Autentica-se na API do Zabbix (login e obtenção do token de sessão). |
| **HTTP Request (Reconhecer evento)** | Communication | Chama o método `event.acknowledge` da API Zabbix, usando o `eventID` e o token de sessão, para confirmar o reconhecimento do problema. |
| **Code (Formata Mensagem Reconhecer)** | Code | Prepara a mensagem original no Telegram para ser editada, **adicionando a linha** "Reconhecido por [Nome do Usuário]". |
| **Telegram (Editar Mensagem Reconhecida)** | Communication | Edita a mensagem no Telegram com o novo texto, confirmando visualmente o reconhecimento para a equipe. |

---

## Fluxo 4: Ação "Ver Histórico"

**Gatilho:** Clique no botão `📝 Ver Histórico` no Telegram.

| Nó | Tipo | Função |
| :--- | :--- | :--- |
| **Switch (Ação do Botão)** | Logic | Filtra o prefixo do *callback_data* (ex: `history:`). |
| **Code (Extrai Trigger ID)** | Code | Extrai o `triggerID` e o `userID` de quem clicou (para resposta privada). |
| **HTTP Request (Zabbix Login - Histórico)** | Communication | Autentica na API do Zabbix. |
| **HTTP Request (Buscar Eventos)** | Communication | Chama `event.get` na API Zabbix para buscar **todos os eventos** (problemas e resoluções) relacionados àquela *trigger* no último período (ex: últimas 24h). |
| **Code (Contar Problemas no Dia)** | Code | Filtra os eventos retornados, calculando o total de problemas que ocorreram no dia de hoje para essa trigger específica. |
| **Code (Formatar historico)** | Code | Monta o relatório de histórico em HTML com os totais calculados. |
| **Telegram (Enviar Histórico)** | Communication | Envia o relatório de histórico **diretamente para o usuário** que solicitou (no chat privado com o bot), mantendo o chat de grupo limpo. |

---

## Fluxo 5: Ação "Pedir Atualização"

**Gatilho:** Clique no botão `🔄 Pedir Atualização` no Telegram.

| Nó | Tipo | Função |
| :--- | :--- | :--- |
| **Switch (Ação do Botão)** | Logic | Filtra o prefixo do *callback_data* (ex: `update:`). |
| **Code (Extrair dados do callback)** | Code | Extrai o `hostName` e o `hostIp` do *callback_data* para o ICMP. |
| **HTTP Request (Zabbix Login - Status)** | Communication | Autentica na API do Zabbix. |
| **HTTP Request (Buscar Status ICMP)** | Communication | Chama `item.get` e `history.get` para buscar o último valor da chave de monitoramento `icmpping` daquele host. |
| **Code (Formata Status do Host)** | Code | Analisa o valor de retorno (`1` para OK, `0` para problema) e o formata como `ONLINE 🟢` ou `OFFLINE 🔴`. |
| **Telegram (Enviar Status)** | Communication | Envia a mensagem de status atual **diretamente para o usuário** que solicitou (no chat privado). |

---

## Fluxo 6: Relatório de Resumo Diário

**Gatilho:** Agendamento (por exemplo, todos os dias às 8h da manhã).

| Nó | Tipo | Função |
| :--- | :--- | :--- |
| **Cron (Agendamento)** | Trigger | Dispara o fluxo no horário pré-definido (ex: `0 8 * * *`). |
| **HTTP Request (Zabbix Login - Resumo)** | Communication | Autentica na API do Zabbix. |
| **HTTP Request (Buscar todos os Eventos)** | Communication | Chama `event.get` para buscar **todos os eventos de hoje** (ou do último período). |
| **Code (Processa e conta problemas)** | Code | Processa a lista de eventos. Calcula: totais de problemas e resoluções, hosts com mais problemas (TOP N) e lista de hosts que permanecem em aberto. |
| **Code (Preparar Mensagem para o Telegram)** | Code | Monta o relatório final em HTML, apresentando os dados de forma clara e profissional. |
| **Telegram (Enviar Relatório)** | Communication | Envia o relatório de Resumo Diário para o canal/grupo de alertas. |
