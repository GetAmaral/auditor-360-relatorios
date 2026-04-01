# Correcao do Classificador — BBBK, Aviacao e Plano de Fix

**Data:** 2026-04-01
**Referencia:** [relatorio-classificador-2026-03-31.md](relatorio-classificador-2026-03-31.md)
**Foco:** Code9, Classificador (Escolher Branch), Prompt Padrao

---

## Parte 1 — Bugs Burger Bug Killer (BBBK)

### O que e a BBBK

A Bugs Burger Bug Killer foi uma empresa de controle de pragas de Miami, fundada por Al Burger. Virou um dos casos mais estudados da Harvard Business School (artigo de Christopher W.L. Hart, HBR 1988: *"The Power of Unconditional Service Guarantees"*).

Num setor visto como commodity barata, Al Burger cobrava **ate 10x mais** que a concorrencia — e tinha as maiores margens de lucro do setor.

### O que eles faziam de diferente

A BBBK oferecia uma **garantia incondicional** sem precedentes:

1. **Voce nao paga ate que TODAS as pragas sejam eliminadas.** Nao reduzidas — eliminadas.
2. **Se voce ficar insatisfeito, recebe reembolso de ate 12 meses** + a BBBK paga outro exterminador por 1 ano.
3. **Se um hospede VER uma praga, a BBBK paga a refeicao/estadia do hospede**, envia carta de desculpas e paga uma visita futura.
4. **Se seu estabelecimento for fechado por pragas, a BBBK paga todas as multas, todo o lucro perdido + US$5.000.**

A concorrencia oferecia "a gente volta e passa o spray de novo".

### Por que sao melhores e mais caros

A garantia nao era marketing — era **ferramenta de gestao**. Ela forcava a empresa a ser excelente porque o custo de falhar era imediato e visivel.

**Principios que faziam a BBBK funcionar:**

| Principio | Como funcionava |
|-----------|----------------|
| **Honestidade radical** | Tecnico que falsificava relatorio era demitido. Esconder problema era mais caro que resolver. |
| **Custo da mentira > custo da verdade** | Mentir num relatorio podia disparar a garantia — custando milhares. Ser honesto custava zero. |
| **Falha como aprendizado** | Cada acionamento da garantia era investigado para encontrar a causa sistemica, nao para punir alguem. |
| **Pagamento premium** | Cobravam 10x mais porque precisavam de margem para fazer o trabalho CERTO. Preco baixo forca atalhos, atalhos disparam garantias, garantias matam o negocio. |
| **Tecnicos bem pagos e treinados** | Tecnico ruim era catastroficamente caro. Investir em gente boa era a opcao mais barata. |
| **Transparencia como vantagem** | A maioria das empresas esconde falhas. A BBBK tornava falhas visiveis e caras — e isso virou sua maior vantagem competitiva. |

**Resultado:** taxa de acionamento da garantia de apenas **0.4% a 1%** da receita — justamente porque a garantia forcava tanta qualidade que as falhas quase nao aconteciam.

### A licao central

> *"Se voce nao esta disposto a garantir seu trabalho incondicionalmente, esta admitindo — ao menos implicitamente — que nao confia na propria qualidade. E se VOCE nao confia, por que o cliente confiaria?"*
> — Al Burger

---

## Parte 2 — Como a Falta de BBBK e Aviacao Nos Afetou (Caso Ryan)

### O que aconteceu

Em 31/03/2026, o usuario Ryan Mendes (advogado) enviou 4 mensagens pedindo para agendar compromissos juridicos. Todas as mensagens tinham quebra de linha (padrao de quem cola texto):

```
"Agendar o compromisso:
Dia 15/04/2026 as 11:30: Prazo MM4 0009432-43.2025.8.16.0035"
```

O sistema respondeu:

```
"Perfeito — vou agendar esse compromisso para 15/04/2026 as 11:30."
```

**Nenhum evento foi criado.** O Ryan saiu da conversa acreditando que 4 compromissos juridicos estavam na agenda dele. Nao estavam.

### Onde falhamos como BBBK

| Principio BBBK | O que a BBBK faria | O que a Total fez |
|----------------|--------------------|--------------------|
| **Honestidade radical** | Nunca diria "eliminado" se nao eliminou | Disse "vou agendar" sem ter agendado |
| **Custo da mentira visivel** | Garantia dispara, paga o cliente | Nenhum custo. Ninguem percebeu. A mentira saiu de graca |
| **Falha como aprendizado** | Investiga causa raiz imediatamente | Nenhum alerta. Nenhum log de falha. Descoberto por acaso |
| **Transparencia** | "Tivemos um problema, vamos resolver" | "Perfeito!" (enquanto nada acontecia) |

**O problema nao e tecnico. E filosofico.** O sistema foi construido de forma que **mentir era o caminho de menor resistencia**. Quando o classificador falhava, o prompt padrao preenchia o silencio com uma resposta agradavel e falsa. Ninguem era alertado. Ninguem pagava o custo.

Na BBBK, isso seria o equivalente a um tecnico assinar "sem pragas" sem nem ter entrado no restaurante. O custo cairia imediatamente na empresa. No nosso sistema, o custo caiu no Ryan — que perdeu prazos juridicos.

### Onde falhamos como Aviacao (Swiss Cheese)

Na aviacao, acidentes nunca sao causados por uma falha isolada. Sao causados por **furos alinhados em multiplas camadas de defesa**. O caso Ryan teve 5 furos alinhados:

```
CAMADA 1 — Extracao (Code9)
  ❌ FURO: JSON.parse falha com newline. Retorna JSON bruto em vez de texto.
  Se tivesse tratado: mensagem chegaria limpa ao classificador.

CAMADA 2 — Classificacao (Escolher Branch)
  ❌ FURO: Expressao N8N lanca excecao. Fallback via || nao executa.
  Se tivesse tratado: classificador receberia mensagem via Code9.

CAMADA 3 — Validacao (AUSENTE)
  ❌ FURO: Nenhum node verifica se MENSAGEM ATUAL chegou vazia.
  Se existisse: teria detectado a falha e impedido o roteamento.

CAMADA 4 — Resposta (Prompt Padrao)
  ❌ FURO: Prompt nao proibe confirmacoes de acoes. IA gera falsa confirmacao.
  Se tivesse protecao: teria dito "nao consegui processar" em vez de "vou agendar".

CAMADA 5 — Monitoramento (AUSENTE)
  ❌ FURO: Nenhum alerta quando o classificador roteia para padrao com msg de acao.
  Se existisse: teria alertado que algo esta errado.
```

Todos os 5 furos se alinharam. O "acidente" aconteceu: **falsa confirmacao entregue ao usuario**.

Na aviacao, a resposta a isso e: **nao basta fechar um furo. Precisa fechar todos, porque da proxima vez os furos podem se alinhar de outro jeito.**

---

## Parte 3 — Escopo do Fix (Code9 + Classificador + Prompt Padrao)

### Visao geral

3 componentes, 3 fixes. Cada um fecha uma camada de defesa.

```
           Code9              Escolher Branch         Prompt Padrao
        (Extracao)            (Classificacao)          (Resposta)
            |                       |                       |
  Fix 1: tratar \n          Fix 2: simplificar       Fix 3: anti-alucinacao
  antes do parse            expressao                + redirecionamentos
            |                       |                       |
    Mensagem limpa  ──>  Classificador recebe  ──>  Se cair em padrao,
    para proximos           mensagem real            NUNCA mente
    nodes
```

---

### Fix 1 — Code9: extractMessageText()

**Node:** Code9
**Workflow:** Premium Main (`tyJ3YAAtSg1UurFj`)

#### Causa

A funcao `extractMessageText()` usa `JSON.parse()` diretamente na string vinda do Redis. Quando a mensagem do usuario tem quebra de linha (muito comum no WhatsApp — colar texto, listas, mensagens estruturadas), o JSON string contem um newline literal. `JSON.parse()` falha porque newline literal e invalido dentro de JSON strings (RFC 8259 exige `\n` como sequencia de escape, nao o caractere real).

Quando `safeParse()` falha, o fallback retorna a string bruta:
```
'{"message_user":"Agendar o compromisso: Dia 15/04/2026..."}'
```
Em vez do texto limpo:
```
"Agendar o compromisso: Dia 15/04/2026..."
```

#### Codigo atual (bugado)

```javascript
function extractMessageText(v) {
  if (!v) return '';
  if (typeof v === 'object' && v !== null) {
    return sanitize(v.message_user ?? v.text ?? '');
  }
  if (typeof v === 'string') {
    const p = safeParse(v);
    if (p && typeof p === 'object' && !Array.isArray(p)) {
      return sanitize(p.message_user ?? p.text ?? '');
    }
    if (typeof p === 'string') {
      const p2 = safeParse(p);
      if (p2 && typeof p2 === 'object') {
        return sanitize(p2.message_user ?? p2.text ?? '');
      }
      return sanitize(p);
    }
    return sanitize(v);
  }
  return '';
}
```

#### Codigo corrigido

```javascript
function extractMessageText(v) {
  if (!v) return '';
  if (typeof v === 'object' && v !== null) {
    return sanitize(v.message_user ?? v.text ?? '');
  }
  if (typeof v === 'string') {
    // 1. Normalizar caracteres de controle ANTES do parse
    //    Newlines literais dentro de JSON strings sao invalidos.
    //    Substituir por sequencias de escape validas.
    const normalized = v
      .replace(/\n/g, '\\n')
      .replace(/\r/g, '\\r')
      .replace(/\t/g, '\\t');

    const p = safeParse(normalized);
    if (p && typeof p === 'object' && !Array.isArray(p)) {
      return sanitize(p.message_user ?? p.text ?? '');
    }

    // 2. Tentar parse do original (caso ja esteja escapado)
    const pOrig = safeParse(v);
    if (pOrig && typeof pOrig === 'object' && !Array.isArray(pOrig)) {
      return sanitize(pOrig.message_user ?? pOrig.text ?? '');
    }
    if (typeof pOrig === 'string') {
      const p2 = safeParse(pOrig);
      if (p2 && typeof p2 === 'object') {
        return sanitize(p2.message_user ?? p2.text ?? '');
      }
      return sanitize(pOrig);
    }

    // 3. Ultimo recurso: extrair via regex
    //    Se o JSON esta quebrado mas tem a estrutura, pegar o valor direto.
    const match = v.match(/"message_user"\s*:\s*"((?:[^"\\]|\\.)*)"/);
    if (match) {
      return sanitize(match[1].replace(/\\n/g, ' ').replace(/\\"/g, '"'));
    }

    return sanitize(v);
  }
  return '';
}
```

#### O que muda

| Cenario | Antes | Depois |
|---------|-------|--------|
| `'{"message_user":"Agendar\nDia 15/04..."}'` | Retorna JSON bruto | Retorna `"Agendar Dia 15/04..."` |
| `'{"message_user":"reuniao 14h"}'` | Funciona | Continua funcionando |
| JSON completamente invalido | Retorna string bruta | Tenta regex, depois string bruta |

---

### Fix 2 — Escolher Branch: Expressao do Classificador

**Node:** Escolher Branch
**Workflow:** Premium Main (`tyJ3YAAtSg1UurFj`)
**Campo:** `text` (prompt template, secao `# INPUT`)

#### Causa

A expressao no prompt template do classificador faz `JSON.parse()` diretamente nos itens do array Lista:

```
{{ $('If19').item.json.Lista.map(m => JSON.parse(m).message_user).join(', ')
   || $('Code9').item.json.mensagem_principal }}
```

Quando `JSON.parse(m)` lanca excecao (newline no JSON), o evaluator de expressoes do N8N **nao avalia o lado direito do `||`**. A expressao inteira retorna string vazia. O classificador recebe `MENSAGEM ATUAL: ""`.

#### Expressao atual (bugada)

```
MENSAGEM ATUAL: "{{ $('If19').item.json.Lista.map(m => JSON.parse(m).message_user).join(', ') || $('Code9').item.json.mensagem_principal }}"
```

#### Expressao corrigida

```
MENSAGEM ATUAL: "{{ $('Code9').item.json.mensagem_principal }}"
```

#### Por que simplificar

1. O Code9 (com o Fix 1 aplicado) ja faz toda a extracao com tratamento de newline, fallbacks e regex.
2. Duplicar a logica de parse na expressao N8N e fragil — expressoes N8N nao suportam try/catch.
3. Uma unica fonte de verdade (`mensagem_principal` do Code9) elimina divergencia entre o que o classificador recebe e o que os outros nodes recebem.
4. **Principio BBBK:** simplicidade reduz pontos de falha. Quanto menos caminhos de codigo, menos furos no Swiss Cheese.

#### Dependencia

**Fix 1 (Code9) DEVE ser aplicado ANTES do Fix 2.** A expressao simplificada depende do Code9 retornando texto limpo.

---

### Fix 3 — Prompt Padrao: Anti-Alucinacao e Redirecionamentos

**Node:** padrao
**Workflow:** Premium Main (`tyJ3YAAtSg1UurFj`)
**Campo:** prompt do sistema

#### Causa

O prompt padrao atual instrui:
```
"NAO executa acoes nem chama tools. Apenas devolve mensagem conversacional."
```

Mas o modelo ignora e gera respostas como:
- "Perfeito — vou agendar esse compromisso"
- "Compromisso atualizado na sua agenda"
- "Evento atualizado! Tudo certo"

Isso acontece porque:
1. A instrucao e **generica demais** — "nao executa acoes" nao e forte o suficiente para impedir o modelo de gerar texto que PARECE confirmacao.
2. O modelo recebe a mensagem do usuario via node `infos` e tenta ser "util" respondendo no contexto do pedido.
3. **Nao existe lista explicita de frases proibidas.**
4. **Nao existe redirecionamento** para quando o usuario pede recursos do sistema (dashboard, planilha, link).

#### Bloco a adicionar no prompt padrao

```
HONESTIDADE RADICAL (REGRA ABSOLUTA — ACIMA DE TODAS AS OUTRAS):

Voce e o modulo CONVERSACIONAL da Total Assistente. Voce NAO tem acesso
a nenhuma ferramenta, nenhuma API, nenhum banco de dados. Voce nao cria,
nao edita, nao exclui, nao busca, nao gera relatorios, nao registra gastos.

Se o usuario pedir qualquer ACAO (agendar, criar, registrar, editar,
excluir, cancelar, buscar, gerar relatorio, enviar planilha), responda:

  "Desculpa, nao consegui processar sua solicitacao agora. Pode tentar
   enviar novamente? Se o problema persistir, tente mandar tudo em uma
   unica mensagem."

FRASES TERMINANTEMENTE PROIBIDAS (nunca usar, em nenhum contexto):
- "vou agendar"
- "agendado"
- "compromisso marcado"
- "registrado"
- "anotado"
- "criado"
- "atualizado"
- "excluido"
- "cancelado"
- "pronto"
- "feito"
- "pode deixar"
- "ja fiz"
- "tudo certo"
- Qualquer variacao que implique que uma acao foi executada

REGRA: se voce NAO chamou uma tool e NAO recebeu confirmacao de sucesso
de volta, voce NAO fez nada. E se NAO fez, NAO diz que fez.

REDIRECIONAMENTOS (quando o usuario pedir acesso ao sistema):
Se a mensagem contiver qualquer uma dessas intencoes:
- "dashboard", "painel", "meu painel"
- "planilha", "minha planilha"
- "site", "link", "acessar", "entrar", "login"
- "quero ver online", "onde vejo", "como acesso"

Responda com o link do sistema:
  "Voce pode acessar tudo pelo nosso site: www.totalassistente.com.br"

Se pedir especificamente sobre o app ou download:
  "Nosso sistema funciona pelo site: www.totalassistente.com.br — da pra
   acessar pelo celular tambem, direto no navegador."
```

#### O que muda

| Cenario | Antes | Depois |
|---------|-------|--------|
| Msg de acao cai no padrao | "Perfeito, vou agendar!" | "Desculpa, nao consegui processar. Pode reenviar?" |
| "Me manda a planilha" | Resposta generica | "Acesse: www.totalassistente.com.br" |
| "Quero ver meu painel" | Resposta generica | "Acesse: www.totalassistente.com.br" |
| "Oi, tudo bem?" | Saudacao normal | Saudacao normal (nao muda) |
| "O que voce faz?" | Explica funcionalidades | Explica funcionalidades (nao muda) |

---

## Parte 4 — Ordem de Aplicacao e Verificacao

### Sequencia

```
PASSO 1 ──> Aplicar Fix 1 (Code9)
              Testar: enviar mensagem com \n via webhook dev
              Verificar: mensagem_principal tem texto limpo (sem JSON bruto)

PASSO 2 ──> Aplicar Fix 2 (Escolher Branch)
              Testar: enviar mensagem com \n via webhook dev
              Verificar: MENSAGEM ATUAL no classificador esta preenchida

PASSO 3 ──> Aplicar Fix 3 (Prompt Padrao)
              Testar: forcar branch padrao com mensagem de acao
              Verificar: IA responde "nao consegui processar" (nunca "vou agendar")

PASSO 4 ──> Rodar bateria completa (24 testes)
              Comparar com relatorio pre-fix (relatorio-classificador-2026-03-31.md)
```

### Criterio de sucesso

| Metrica | Pre-fix | Pos-fix esperado |
|---------|---------|------------------|
| Testes PASS | 8/24 (33%) | 18+/24 (75%+) |
| Falsas confirmacoes | 7/24 (29%) | **0/24 (0%)** |
| Sem resposta | 6/24 (25%) | Investigar separado (debounce) |

### O que este fix NAO resolve (backlog)

| Bug | Status | Motivo |
|-----|--------|--------|
| Mensagens sem resposta (debounce) | Backlog | Causa diferente — nao e classificador |
| Eventos recorrentes como avulsos | Backlog | Bug no prompt de criacao recorrente |
| Deteccao duplicata (A1) | Backlog | Bug no fluxo de criacao |

---

## Parte 5 — Principio Operacional (BBBK + Aviacao)

A partir desta correcao, o Total Assistente opera sob dois principios:

### Principio 1 — Honestidade Radical (BBBK)

> A Total NUNCA mente. Se nao fez, nao diz que fez. Se nao sabe, diz
> que nao sabe. Se nao consegue, orienta o usuario.
>
> O custo de ser honesto (pedir reenvio) e infinitamente menor que o
> custo de mentir (usuario perde compromisso juridico confiando em
> confirmacao falsa).

### Principio 2 — Defesa em Profundidade (Swiss Cheese)

> Nenhuma camada sozinha e confiavel. O sistema tem 3 camadas de defesa:
>
> 1. **Prevencao** — Code9 trata newlines antes que causem problema
> 2. **Deteccao** — Classificador recebe mensagem real, nao string vazia
> 3. **Contencao** — Prompt padrao nunca confirma acao que nao executou
>
> Se a Camada 1 falhar, a Camada 2 pega. Se a Camada 2 falhar, a
> Camada 3 impede a mentira. Nenhum furo isolado causa dano ao usuario.

---

*Documento gerado em 2026-04-01 | Metodologia: BBBK (Harvard, 1988) + Aviation Safety (Swiss Cheese, HFACS)*
