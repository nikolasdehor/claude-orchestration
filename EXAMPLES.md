# Examples

Exemplos antes/depois para os 5 princípios de orquestração. Cada exemplo mostra o que orquestradores costumam fazer errado e como consertar.

Pt-BR | [English](#examples-english)

---

## 1. Delegate Over Execute

### O orquestrador executa "só pra ser rápido"

**Pedido do usuário:** "Dá uma olhada no erro que apareceu no log do deploy."

**Errado (orquestrador lê direto)**

```
[Opus chama Read em /var/log/deploy.log]
[Opus chama Grep pra filtrar o erro]
[Opus chama Read em 3 arquivos pra contextualizar]
[Opus finalmente responde ao usuário]
```

O orquestrador gastou 4 tool calls e boa parte do contexto carregando texto bruto que ele não vai precisar depois. Quando a próxima decisão chegar, esse contexto vai fazer falta.

**Certo (delega pra um executor)**

```
1. TeamCreate
2. Agent devops com briefing curto:
   "Leia /var/log/deploy.log, filtre erros das ultimas 2 horas,
    me retorne apenas: tipo de erro, linha, hipotese de causa.
    Nao cole o log inteiro."
3. Recebe resumo em 3 linhas
4. Decide com base no resumo
5. TeamDelete
```

O orquestrador nunca leu o log. Consumiu 3 linhas em vez de 3 mil. Contexto preservado pra próxima decisão.

### "Deixa eu corrigir, é só uma linha"

**Pedido do usuário:** "Esse typo no README."

**Errado**

```
[Opus chama Edit direto. 1 linha alterada.]
```

Parece inocente. Mas o hábito vaza: da próxima vez é "deixa eu editar o arquivo X que é só uma função". Do mesmo jeito, "deixa eu rodar esse comando rápido". Em duas horas o orquestrador virou o executor.

**Certo**

Delegue mesmo assim. Um agent haiku resolve em 5 segundos com briefing de uma linha. O ganho não é a velocidade, é a disciplina.

---

## 2. Teams Are Mortal

### Time órfão rodando de madrugada

**Errado**

```
[TeamCreate: time-frontend]
[Agent frontend-dev faz a task]
[Retorna com "feito"]
[Orquestrador: "ok, obrigado"]
[Time continua vivo, em idle loop]
[3 horas depois, usuario volta, nota processo consumindo memoria]
```

Isso acontece porque "talvez eu precise de novo". Mas quando precisa de novo, o estado do time já está desalinhado e é melhor criar do zero.

**Certo**

```
[TeamCreate: time-frontend]
[Agent frontend-dev faz a task]
[Orquestrador roda git diff pra verificar]
[Orquestrador: SendMessage shutdown_request]
[TeamDelete]
[3 horas depois, se precisar, TeamCreate novo]
```

### "Vou reaproveitar o time"

**Errado**

```
[Time "backend-fix" criado pra bug de auth]
[Bug corrigido, time continua]
[2 dias depois, pedido novo de feature: "vou usar o mesmo time"]
```

O contexto do time está saturado de discussão sobre auth. A feature nova vai herdar ruído, e o briefing vai ficar grande demais tentando compensar.

**Certo**

Time novo pra objetivo novo. Briefing limpo, contexto zerado, executor foca.

---

## 3. Trust But Verify

### "Feito" sem verificação

**Pedido:** "Adicione validação de email no formulário de signup."

**Errado**

```
Teammate: "Feito. Adicionei validacao de regex no handler."
Orquestrador: "Otimo! [responde ao usuario que esta pronto]"
```

Dois dias depois o usuário reporta que a validação não está funcionando. O teammate tinha adicionado sim, mas numa branch errada, ou num componente que não é mais usado.

**Certo**

```
Teammate: "Feito. Adicionei validacao no handleSubmit do SignupForm.tsx."
Orquestrador: delega verificador separado
  "Abra o signup no navegador, digite email invalido,
   confirme que a mensagem de erro aparece.
   Retorne screenshot ou confirmacao textual."
Verificador: "Confirmado, erro aparece como esperado."
Orquestrador: responde ao usuario que esta pronto
```

Custou uma delegação a mais. Evitou retrabalho e perda de confiança.

### Relatório bonito, diff vazio

**Errado**

```
Teammate: "Refatorei o modulo, removi 3 duplicacoes, melhorei a legibilidade."
Orquestrador: aceita e segue
```

**Certo**

```
Teammate: mesmo relatorio
Orquestrador: roda git diff antes de aceitar
Descobre que so 1 duplicacao foi removida, e 2 funcoes novas foram criadas sem serem usadas
SendMessage: "Remova as 2 funcoes nao usadas e faca a outra duplicacao."
```

O relatório descreve a intenção. O diff descreve o resultado.

---

## 4. Surgical Delegation

### Briefing vago

**Pedido do usuário:** "Melhore o site."

**Errado (repassar literal)**

```
Agent frontend-dev: "Melhore o site."
```

O teammate vai inventar. Vai mexer em cores, espaçamento, fontes, componentes não pedidos, reescrever o hero. Diff gigante, maioria descartável.

**Certo (decompor antes de delegar)**

```
Orquestrador pergunta ao usuario qual parte esta incomodando
Resposta: "o hero esta muito vazio no mobile"

Agent frontend-dev recebe briefing cirurgico:
  Objetivo: ajustar o hero em viewports menores que 768px
  Constraints:
    - nao alterar desktop
    - manter a fonte atual
    - arquivo: src/components/Hero.tsx
  Regras: pt-BR, sem em dashes, match estilo existente
  Entrega: diff do Hero.tsx + screenshot do before/after no mobile
```

Mesmo agente, mesmo modelo, resultado completamente diferente.

### Briefing gigante

**Errado**

```
Agent recebe: historico completo da conversa + decisoes antigas
              + 4 arquivos colados + 2 links + "e lembra daquilo que falamos?"
```

O executor gasta contexto processando coisa que não precisa. E não sabe qual parte priorizar.

**Certo**

```
Agent recebe:
  Objetivo (1 linha)
  3-5 bullets de constraint
  Caminho pra arquivo X, linhas Y-Z
  Formato de entrega
```

Se o teammate precisar de mais, ele pergunta. Melhor do que pre-carregar tudo.

---

## 5. Language Discipline

### Em dash vaza no commit

**Errado**

```
Teammate recebe: "faca o commit da mudanca"
Commit gerado: "feat: add validation — handles edge cases and trims whitespace"
```

O em dash (U+2014) entrou sem que ninguém notasse. O usuário vê "edge cases and trims whitespace" e reconhece na hora: AI escreveu isso.

**Certo**

```
Briefing inclui explicito:
  - pt-BR com acentos corretos
  - NUNCA use em dashes U+2014 ou U+2013
  - Use hifen simples (-) ou virgula
  - Autor: Nikolas de Hor (lowercase "de")
  - Sem "Co-Authored-By" ou referencia a AI

Antes de aceitar o commit:
  grep -P '[\x{2014}\x{2013}]' commit-message
  Se retornar algo, rejeita e pede correcao.
```

### Nome do autor errado no README

**Errado**

```
Teammate gera: "Criado por nikolas de Hor em Goiania."
```

Dois erros: "nikolas" sem maiúscula no começo ("Nikolas" é nome próprio), "Goiania" sem acento.

**Certo**

```
Briefing: "Autor: Nikolas de Hor. Cidade: Goiania escreve com acento, Goiânia."
Verificador: grep "Goiania\|Goiânia" README.md
Se aparecer "Goiania" sem acento, rejeita.
```

Dez segundos de verificação, reputação do repo preservada.

---

## Resumo anti-padrões

| Princípio | Anti-padrão | Correção |
|-----------|-------------|----------|
| Delegate Over Execute | Orquestrador lê 1 arquivo "pra ser rápido" | Delega mesmo tarefas pequenas |
| Teams Are Mortal | Deixa time aberto "pro caso de precisar" | TeamDelete imediato, time novo depois |
| Trust But Verify | Aceita "feito" sem conferir diff | Sempre confere antes de reportar ao usuário |
| Surgical Delegation | Repassa pedido vago ou cola toda a conversa | Objetivo + constraints + regras + formato |
| Language Discipline | Assume que teammate herdou estilo | Regras de idioma em todo briefing + verificação grep |

---

# Examples (English)

Before/after examples for the 5 orchestration principles. Each one shows what orchestrators commonly get wrong and how to fix it.

## 1. Delegate Over Execute

### "Let me just do it fast"

**Request:** "Check the error that showed up in the deploy log."

**Wrong (orchestrator reads directly)**

```
[Opus calls Read on /var/log/deploy.log]
[Opus calls Grep to filter the error]
[Opus calls Read on 3 files for context]
[Opus finally answers the user]
```

The orchestrator spent 4 tool calls and a good chunk of context loading raw text it will not need later. The next decision will feel that loss.

**Right (delegate to a worker)**

```
1. TeamCreate
2. Agent devops with a short briefing:
   "Read /var/log/deploy.log, filter errors from the last 2 hours,
    return only: error type, line, root cause hypothesis.
    Do not paste the full log."
3. Receives 3-line summary
4. Decides based on the summary
5. TeamDelete
```

The orchestrator never read the log. Consumed 3 lines instead of 3,000. Context preserved.

### "It is only one line"

**Request:** "Fix this typo in the README."

**Wrong**

```
[Opus calls Edit directly. 1 line changed.]
```

Looks innocent. But the habit leaks: next time it is "let me just edit this small function". Then "let me just run this quick command". In two hours the orchestrator became the worker.

**Right**

Delegate anyway. A haiku agent solves it in 5 seconds with a one-line briefing. The gain is not speed, it is discipline.

---

## 2. Teams Are Mortal

### Orphan team running overnight

**Wrong**

```
[TeamCreate: team-frontend]
[Agent frontend-dev does the task]
[Returns "done"]
[Orchestrator: "ok, thanks"]
[Team stays alive, idle loop]
[3 hours later, user comes back, notices process burning memory]
```

This happens because "maybe I will need it again". But when the need comes, the team's state is already misaligned and it is better to create a fresh one.

**Right**

```
[TeamCreate: team-frontend]
[Agent frontend-dev does the task]
[Orchestrator runs git diff to verify]
[Orchestrator: SendMessage shutdown_request]
[TeamDelete]
[3 hours later, if needed, fresh TeamCreate]
```

### "I will reuse the team"

**Wrong**

```
[Team "backend-fix" created for an auth bug]
[Bug fixed, team stays alive]
[2 days later, new feature request: "I will use the same team"]
```

The team context is saturated with auth discussion. The new feature inherits noise, and the briefing balloons to compensate.

**Right**

New team for a new objective. Clean briefing, zeroed context, focused worker.

---

## 3. Trust But Verify

### "Done" without verification

**Request:** "Add email validation to the signup form."

**Wrong**

```
Teammate: "Done. Added regex validation in the handler."
Orchestrator: "Great! [tells user it is ready]"
```

Two days later the user reports the validation is not working. The teammate did add it, but in the wrong branch, or in a component no longer used.

**Right**

```
Teammate: "Done. Added validation in SignupForm.tsx handleSubmit."
Orchestrator: delegates a separate verifier
  "Open signup in the browser, type an invalid email,
   confirm the error message appears.
   Return screenshot or textual confirmation."
Verifier: "Confirmed, error appears as expected."
Orchestrator: tells the user it is ready
```

One extra delegation. Prevented rework and trust loss.

### Nice report, empty diff

**Wrong**

```
Teammate: "Refactored the module, removed 3 duplications, improved readability."
Orchestrator: accepts and moves on
```

**Right**

```
Teammate: same report
Orchestrator: runs git diff before accepting
Discovers only 1 duplication was removed, and 2 unused functions were added
SendMessage: "Remove the 2 unused functions and handle the remaining duplication."
```

The report describes intent. The diff describes outcome.

---

## 4. Surgical Delegation

### Vague briefing

**Request:** "Improve the site."

**Wrong (pass through as-is)**

```
Agent frontend-dev: "Improve the site."
```

The teammate will make things up. Colors, spacing, fonts, unrequested components, a full hero rewrite. Giant diff, mostly throwaway.

**Right (decompose before delegating)**

```
Orchestrator asks the user which part is bothering them
Answer: "the hero feels empty on mobile"

Agent frontend-dev gets a surgical briefing:
  Objective: adjust the hero on viewports below 768px
  Constraints:
    - do not change desktop
    - keep the current font
    - file: src/components/Hero.tsx
  Rules: match existing style, no em dashes
  Delivery: Hero.tsx diff + before/after mobile screenshot
```

Same agent, same model, completely different result.

### Bloated briefing

**Wrong**

```
Agent receives: full conversation history + old decisions
               + 4 pasted files + 2 links + "and remember what we talked about?"
```

The worker burns context processing things it does not need. And does not know what to prioritize.

**Right**

```
Agent receives:
  Objective (1 line)
  3-5 bullet constraints
  Path to file X, lines Y-Z
  Delivery format
```

If the teammate needs more, they will ask. Better than pre-loading everything.

---

## 5. Language Discipline

### Em dash leaks into the commit

**Wrong**

```
Teammate receives: "commit the change"
Generated commit: "feat: add validation — handles edge cases and trims whitespace"
```

The em dash (U+2014) slipped through unnoticed. The user sees "edge cases and trims whitespace" and spots it instantly: AI wrote this.

**Right**

```
Briefing includes explicitly:
  - Correct English or pt-BR with accents, per user preference
  - NEVER use em dashes U+2014 or U+2013
  - Use single hyphen (-) or comma
  - Author: Nikolas de Hor (lowercase "de")
  - No "Co-Authored-By" or AI reference

Before accepting the commit:
  grep -P '[\x{2014}\x{2013}]' commit-message
  If it returns anything, reject and request a fix.
```

### Wrong author name in the README

**Wrong**

```
Teammate generates: "Built by nikolas de Hor in Goiania."
```

Two errors: "nikolas" lowercase at the start ("Nikolas" is a proper noun), "Goiania" missing its accent.

**Right**

```
Briefing: "Author: Nikolas de Hor. City: Goiania is written with an accent, Goiânia."
Verifier: grep "Goiania\|Goiânia" README.md
If "Goiania" appears without accent, reject.
```

Ten seconds of verification, repo reputation preserved.

---

## Anti-pattern summary

| Principle | Anti-pattern | Fix |
|-----------|--------------|-----|
| Delegate Over Execute | Orchestrator reads 1 file "to be fast" | Delegate even small tasks |
| Teams Are Mortal | Leaves team open "just in case" | Immediate TeamDelete, fresh team later |
| Trust But Verify | Accepts "done" without checking diff | Always verify before reporting to user |
| Surgical Delegation | Passes vague request or dumps full chat | Objective + constraints + rules + format |
| Language Discipline | Assumes teammate inherited style | Language rules in every briefing + grep verification |
