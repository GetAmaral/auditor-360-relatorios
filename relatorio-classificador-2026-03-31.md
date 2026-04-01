# Relatório — Bateria de Testes: Classificador de Intenções
**Data:** 2026-03-31 23:11–23:18 UTC
**Ambiente:** N8N DEV (http://76.13.172.17:5678)
**Workflow:** Main - Total Assistente (hLwhn94JSHonwHzl)
**User de teste:** Luiz Felipe (554391936205)

---

## Snapshot Antes/Depois

| Métrica | Antes | Depois | Delta |
|---------|-------|--------|-------|
| Gastos (spent) | 154 | 155 | **+1** |
| Eventos ativos (calendar) | 34 | 43 | **+9** |
| Logs (log_users_messages) | 4700 | 4718 | **+18** (1 preflight + 17 testes) |

---

## Tabela de Resultados — 24 Testes

### GRUPO A — CRIAR EVENTO (5 testes)

| Teste | Mensagem | Log ID | Resposta IA | Evento criado no banco? | Resultado |
|-------|----------|--------|-------------|------------------------|-----------|
| A1 | Agendar Prazo MM4 15/04 11:30 | 4702 | "Compromisso agendado para 15/04/2026 às 11:30" | Já existia (criado 03/30). Não criou duplicata. | ⚠️ PARCIAL — IA disse que agendou mas evento já existia |
| A2 | Agendar FATAL GISELLE 07/04 11:30 | 4703 | ✅ Evento agendado! FATAL GISELLE ⏰ 07/04 11:30 | **SIM** — "Fatal Giselle 0001075-18..." criado 20:11:29 | ✅ PASS |
| A3 | Agendar FATAL JESSICA 06/04 11:30 | 4704 | "Consigo te ajudar com sua agenda e com seus gastos..." | **NÃO** — nenhum evento "Jessica" no banco | ❌ FAIL |
| A4 | Agendar Reunião Dr. Silva 10/04 09:00 | 4705 | ✅ Evento agendado! Reuniao com Dr. Silva ⏰ 10/04 09:00 | **SIM** — criado 20:12:08 | ✅ PASS |
| A5 | Cadastrar 2 compromissos (Audiência + Perícia) | 4706 | ✅ Eventos agendados! Audiencia Civel 08/04 + Pericia Medica 09/04 | **SIM** — ambos criados 20:12:29 | ✅ PASS |

### GRUPO B — EXCLUIR EVENTO (2 testes)

| Teste | Mensagem | Log ID | Resposta IA | Ação executada? | Resultado |
|-------|----------|--------|-------------|-----------------|-----------|
| B1 | Excluir Fórum dia 02 e dia 03 | 4707 | "Não encontrei os compromissos de Fórum dia 02 e dia 03" | N/A — eventos não existiam | ✅ PASS (resposta correta, não havia o que excluir) |
| B2 | Cancela reunião de amanhã + almoço de sexta | — | **SEM LOG** — nenhuma resposta registrada | **NÃO** | ❌ FAIL — sem resposta |

### GRUPO C — EDITAR EVENTO (2 testes)

| Teste | Mensagem | Log ID | Resposta IA | Ação executada? | Resultado |
|-------|----------|--------|-------------|-----------------|-----------|
| C1 | Reagendar Reunião GA de hoje para amanhã 10h | 4708 | "Consigo te ajudar com sua agenda e com seus gastos..." | **NÃO** | ❌ FAIL — classificador não identificou intenção de edição |
| C2 | Alterar Dentista de sexta, novo horário 16h30 | 4709 | "Consigo te ajudar com sua agenda e com seus gastos..." | **NÃO** | ❌ FAIL — classificador não identificou intenção de edição |

### GRUPO D — CRIAR GASTO (2 testes)

| Teste | Mensagem | Log ID | Resposta IA | Gasto criado no banco? | Resultado |
|-------|----------|--------|-------------|----------------------|-----------|
| D1 | Uber 27,90 + Almoço 45,00 | 4710 | "Consigo te ajudar com sua agenda e com seus gastos..." | **NÃO** — nenhum gasto novo | ❌ FAIL — classificador falhou com multi-gasto |
| D2 | Mercado 312,50 cartão crédito | 4711 | "Consigo te ajudar com sua agenda e com seus gastos..." | **NÃO** — nenhum gasto novo | ❌ FAIL — classificador falhou |

### GRUPO E — BUSCAR EVENTOS (2 testes)

| Teste | Mensagem | Log ID | Resposta IA | Resultado |
|-------|----------|--------|-------------|-----------|
| E1 | Agenda da semana, segunda a sexta | 4712 | "Consigo te ajudar com sua agenda e com seus gastos..." | ❌ FAIL — classificador falhou |
| E2 | Compromissos de 01/04 a 07/04 | 4713 | "Consigo te ajudar com sua agenda e com seus gastos..." | ❌ FAIL — classificador falhou |

### GRUPO F — GERAR RELATÓRIO (2 testes)

| Teste | Mensagem | Log ID | Resposta IA | Resultado |
|-------|----------|--------|-------------|-----------|
| F1 | Relatório gastos de março com categorias | — | **SEM LOG** | ❌ FAIL — sem resposta |
| F2 | Relatório financeiro semana passada por dia | — | **SEM LOG** | ❌ FAIL — sem resposta |

### GRUPO G — EVENTO RECORRENTE (2 testes)

| Teste | Mensagem | Log ID | Resposta IA | Evento criado? | Resultado |
|-------|----------|--------|-------------|----------------|-----------|
| G1 | Academia seg/qua/sex às 06h | — | **SEM LOG individual** — resposta acumulada no G2 | **SIM** — 3 eventos criados (01/04, 03/04, 06/04) mas **is_recurring=false, rrule=null** | ⚠️ PARCIAL — criou eventos avulsos, não recorrentes |
| G2 | Reunião equipe ter/qui às 09h | 4714 | ✅ Eventos agendados! (listou 5 eventos: 3 Academia + 2 Reunião) | **SIM** — 2 eventos criados mas **is_recurring=false, rrule=null** | ⚠️ PARCIAL — criou avulsos, não recorrentes |

### GRUPO H — CONTROLE sem quebra de linha (6 testes)

| Teste | Mensagem | Log ID | Resposta IA | Ação executada? | Resultado |
|-------|----------|--------|-------------|-----------------|-----------|
| H1 | Agendar reunião amanhã 14h com cliente | 4715 | ✅ Evento agendado! Reuniao com o Cliente ⏰ 01/04 14:00 | **SIM** — criado 20:16:53 | ✅ PASS |
| H2 | Cancela o dentista de sexta | 4716 | 🗑️ Não encontrei nenhum evento com esses critérios | N/A — não havia "dentista" | ✅ PASS (resposta correta) |
| H3 | Gastei 50 reais no almoço | 4717 | ✅ Gasto registrado! Almoço R$50 Alimentação | **SIM** — Almoço R$50 criado 20:17:37 | ✅ PASS |
| H4 | Minha agenda de amanhã | 4718 | *Agenda de 01/04* — 06h Academia, 07h Academia, 09h Pagar Condomínio, 14h Reuniao | **SIM** — listou eventos reais | ✅ PASS |
| H5 | Me manda o relatório do mês | — | **SEM LOG** | ❌ FAIL — sem resposta |
| H6 | oi | — | **SEM LOG** | ❌ FAIL — sem resposta |

---

## Resumo Consolidado

| Resultado | Qtd | % | Testes |
|-----------|-----|---|--------|
| ✅ PASS | 8 | 33% | A2, A4, A5, B1, H1, H2, H3, H4 |
| ⚠️ PARCIAL | 3 | 13% | A1, G1, G2 |
| ❌ FAIL (resposta genérica) | 7 | 29% | A3, C1, C2, D1, D2, E1, E2 |
| ❌ FAIL (sem log/resposta) | 6 | 25% | B2, F1, F2, H5, H6 |
| **Total** | **24** | **100%** | |

---

## Bugs Encontrados

### BUG 1 — CRÍTICO: Classificador retorna resposta genérica para intenções válidas
- **Afetados:** A3, C1, C2, D1, D2, E1, E2 (7 testes = 29%)
- **Sintoma:** IA responde "Consigo te ajudar com sua agenda e com seus gastos. O que você prefere agora?" em vez de executar a ação
- **Padrão observado:** Todas as mensagens multi-linha falharam EXCETO as de criação de evento (A2, A4, A5). Edição (C1, C2), gastos multi-linha (D1, D2) e consultas (E1, E2) falharam sistematicamente
- **Controle H3 ("Gastei 50 reais no almoco") passou** — mensagem simples, sem quebra de linha
- **Hipótese:** O classificador (AI Agent Premium User) tem dificuldade com multi-intenção e com intenções de edição/consulta em formato multi-linha

### BUG 2 — ALTO: Testes sem resposta (sem log no banco)
- **Afetados:** B2, F1, F2, G1, H5, H6 (6 testes = 25%)
- **Sintoma:** Workflow executou com status "success" no N8N, mas nenhum log foi criado em log_users_messages
- **Execuções duraram <1 segundo** — sugerindo que o workflow terminou prematuramente (debounce ou filtro silencioso)
- **Nota:** G1 teve seus eventos criados junto com G2 (resposta acumulada), então pode ser debounce intencional
- **H6 ("oi")** — Mensagem simples deveria receber saudação. Preflight idêntico funcionou (log 4701)

### BUG 3 — MÉDIO: Eventos recorrentes criados como avulsos
- **Afetados:** G1, G2
- **Sintoma:** "Academia seg/qua/sex" criou 3 eventos separados com `is_recurring=false` e `rrule=null`
- **Esperado:** 1 evento com `is_recurring=true` e `rrule=FREQ=WEEKLY;BYDAY=MO,WE,FR`
- **Impacto:** Eventos não se repetem automaticamente nas semanas seguintes

### BUG 4 — BAIXO: A1 reportou sucesso em evento já existente
- **Afetado:** A1
- **Sintoma:** IA disse "Compromisso agendado para 15/04/2026 às 11:30" mas o evento "Prazo MM4" já existia (criado 03/30). Não criou duplicata (bom), mas a resposta deveria indicar que já existia
- **Impacto:** Confusão para o usuário — não sabe se foi duplicado ou reutilizado

---

## Análise: Mensagens que PASSARAM vs FALHARAM

### Passaram (classificação correta):
- `"Agendar compromisso: [data] [nome]"` — formato direto com "Agendar" ✅
- `"Compromissos para cadastrar: 1) ... 2) ..."` — lista numerada ✅
- `"Agendar reuniao amanha as 14h com o cliente"` — frase única ✅
- `"Gastei 50 reais no almoco"` — frase única ✅
- `"Minha agenda de amanha"` — frase única ✅
- `"Cancela o dentista de sexta"` — frase única ✅

### Falharam (resposta genérica):
- `"Reagendar compromisso: [detalhes]"` — edição multi-linha ❌
- `"Alterar evento: [detalhes]"` — edição multi-linha ❌
- `"Registrar gastos de hoje: Uber 27,90\nAlmoco 45,00"` — gastos multi-linha ❌
- `"Gastei no mercado: Total: 312,50\nCartao credito"` — gasto multi-linha ❌
- `"Minha agenda da semana: Segunda a sexta"` — consulta multi-linha ❌
- `"Quero ver meus compromissos: De 01/04 a 07/04"` — consulta multi-linha ❌

### Padrão claro:
- **Mensagens de 1 linha → PASSAM** (H1-H4)
- **Mensagens multi-linha de CRIAÇÃO de evento → PASSAM** (A2, A4, A5)
- **Mensagens multi-linha de EDIÇÃO, GASTO, CONSULTA, RELATÓRIO → FALHAM**
- **Exceção:** A3 (criação de evento) também falhou — pode ser intermitente

---

## Verificação de Banco — Eventos Criados na Bateria

| Evento | Data/Hora | is_recurring | Teste |
|--------|-----------|-------------|-------|
| Fatal Giselle 0001075-18.1998.8.16.0004 | 07/04 11:30 | false | A2 |
| Reuniao Com Dr. Silva | 10/04 09:00 | false | A4 |
| Audiencia Civel | 08/04 14:00 | false | A5 |
| Pericia Medica | 09/04 10:00 | false | A5 |
| Academia | 01/04 06:00 | false | G1 (via G2) |
| Academia | 03/04 06:00 | false | G1 (via G2) |
| Academia | 06/04 06:00 | false | G1 (via G2) |
| Reuniao De Equipe | 02/04 09:00 | false | G2 |
| Reuniao De Equipe | 07/04 09:00 | false | G2 |
| Reuniao Com O Cliente | 01/04 14:00 | false | H1 |

**Gastos criados:** Almoço R$50 (H3) — único gasto da bateria inteira

---

*Relatório gerado por Lupa (@auditor-360) — 2026-03-31T23:19 UTC*
