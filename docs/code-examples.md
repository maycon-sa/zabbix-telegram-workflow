# 💻 Detalhamento dos Nós de Código (Code Nodes)

Este documento explica o que cada bloco de código (Code Node) faz no meu workflow do n8n, facilitando a manutenção e a compreensão da lógica por trás das interações com Zabbix e Telegram.

---

## Fluxo Principal de Alerta (Problema)

### 1. Code (Formatar Mensagem)

**O que eu faço:** Sou responsável por pegar os dados do Zabbix e montar a mensagem de alerta em **HTML** para que fique bonita no Telegram.

**Minha lógica:**
1. Crio uma função de segurança (`escapeHtml`) para evitar que caracteres especiais quebrem o formato HTML da mensagem.
2. Construo o corpo da mensagem (`body`) com todas as informações do alerta (Host, Trigger, Item, Severidade).
3. **Crio os botões de ação** (`inline_keyboard`) e defino o `callback_data` para cada um:
    * `ack:{event_id}:{trigger_id}` (Para o Reconhecimento)
    * `history:{trigger_id}` (Para o Histórico)
    * `update:{host_name}:{host_ip}` (Para o Status Atual)

---

## Fluxo de Resolução

### 2. Code (Formata Mensagem Resolução Editar)

**O que eu faço:** Se a resolução de um problema chegar, eu pego a mensagem original do alerta e a **edito** para sinalizar que o problema acabou.

**Minha lógica:**
1. Acesso os dados da mensagem original (enviada no momento do problema).
2. Uso os dados de *recovery* (hora e data da resolução) que vieram do Zabbix.
3. Formato o novo texto com a tag `✅✨🎉 PROBLEMA RESOLVIDO!`.
4. Retorno a estrutura JSON para o nó `editMessageText` do Telegram, **removendo todos os botões** para "fechar" o alerta.

### 3. Code (Formata Mensagem Resolução)1

**O que eu faço:** Eu entro em ação como um **plano B**. Se, por algum motivo, o alerta de problema original não foi enviado/encontrado no Telegram, eu formato uma **nova mensagem** para notificar a resolução.

**Minha lógica:**
1. Formato o texto de resolução da mesma forma que o nó anterior.
2. Retorno a estrutura JSON para o nó `sendMessage` (envio de nova mensagem) do Telegram.

---

## Fluxos de Interação por Botão (Telegram Trigger)

### 4. Code (Extrair Callback_data)

**O que eu faço:** Eu sou ativado quando alguém clica no botão **"✅ Reconhecer Problema"**. Meu objetivo é extrair os dados necessários para reconhecer o evento no Zabbix e atualizar a mensagem no Telegram.

**Minha lógica:**
1. Pego o `callback_query.data` (ex: `ack:12345:67890`) e separo os campos: `eventId` e `triggerId`.
2. Pego o `first_name` e `last_name` do usuário que clicou para montar o `fullName`.

### 5. Code (Formata Mensagem Reconhecer)

**O que eu faço:** Pego o texto da mensagem original no Telegram e adiciono uma linha de **Reconhecimento**.

**Minha lógica:**
1. Resgato o texto original da mensagem do Telegram.
2. Adiciono uma linha nova: `✅ Reconhecido por [Nome do Usuário] às [Hora Atual]`.
3. **Mantenho os botões de ação** intactos para que a equipe possa continuar usando as funções de Histórico e Atualização.
4. Envio a nova estrutura para o nó de edição de mensagem.

### 6. Code (Extrai Trigger ID)

**O que eu faço:** Sou ativado pelo clique no botão **"📝 Ver Histórico"**. Meu trabalho é apenas extrair o ID da trigger para que o fluxo possa buscar o histórico no Zabbix.

**Minha lógica:**
1. Extraio o `triggerId` do `callback_data` (`history:12345`).
2. Guardo o `chat_id` e `message_id` para fins de referência futura.

### 7. Code (Contar Problemas no Dia)

**O que eu faço:** Eu processo a lista de eventos de uma trigger específica que o Zabbix me retornou. Meu objetivo é filtrar e contar quantos problemas ocorreram **só hoje**.

**Minha lógica:**
1. Defino o `startOfTodayTimestamp` (meia-noite de hoje em Unix).
2. Filtro o array de eventos (`allEvents`) para manter apenas aqueles cujo *timestamp* (`event.clock`) é maior ou igual ao início do dia.
3. O resultado final é a contagem (`problemCount`) e uma lista dos horários em que os problemas ocorreram.

### 8. Code (Formatar historico)

**O que eu faço:** Monto a resposta final do histórico de uma trigger específica para enviar ao usuário no chat privado do bot.

**Minha lógica:**
1. Acesso as informações detalhadas da trigger e a contagem de problemas que foi calculada no nó anterior.
2. Formato um texto conciso e informativo em HTML, mostrando a descrição da Trigger, Status, Prioridade e o total de ocorrências do dia.

### 9. Code (Extrair dados do callback)

**O que eu faço:** Sou ativado pelo clique em **"🔄 Pedir Atualização"**. Eu extraio o nome e IP do host para que o fluxo possa fazer um check rápido de ICMP no Zabbix.

**Minha lógica:**
1. Extraio `hostName` e `hostIp` do `callback_data` (`update:HostA:192.168.1.1`).
2. Armazeno o `userId` de quem solicitou a atualização para garantir que a resposta seja enviada no chat privado correto.

### 10. Code (Formata Status do Host)

**O que eu faço:** Pego o resultado do ICMP que veio do Zabbix e formato a resposta de status para o usuário.

**Minha lógica:**
1. Verifico o `lastvalue` do ICMP: se for "1", o status é **ONLINE 🟢**; se for diferente, é **OFFLINE 🔴**.
2. Monto a mensagem HTML simples e a preparo para envio ao `chat_id` do usuário que clicou.

---

## Fluxo de Resumo Diário (Schedule Trigger)

### 11. Code (Processa e conta problemas)

**O que eu faço:** Eu analiso **todos** os eventos do dia retornados pelo Zabbix para montar o resumo estatístico.

**Minha lógica:**
1. Itero sobre todos os eventos para contar `problems` (valor 1) e `resolved` (valor 0) para cada host.
2. **Ordeno** os hosts pelo número de problemas.
3. Gero a lista dos **TOP 5 hosts** mais afetados.
4. Gero a lista de **Hosts em aberto** (onde `problems > resolved`).
5. Calculo os totais gerais e retorno os dados.

### 12. Code (Preparar Mensagem para o Telegram)

**O que eu faço:** Pego os dados processados no nó anterior e monto o relatório final para o Resumo Diário.

**Minha lógica:**
1. Uso os totais (`totalProblems`, `totalResolved`) e as listas formatadas (`topHosts`, `openHosts`).
2. Construo um cabeçalho e corpo ricos em HTML, garantindo que o relatório seja claro e destaque o que está pendente.
