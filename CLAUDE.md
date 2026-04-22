# CLAUDE.md

Diretrizes de orquestração multi-agent para Claude Code. Destiladas de 6 meses rodando times de agentes em produção (OpenClaw, swarms customizados, agent teams persistentes). Junte com instruções específicas do seu projeto.

**Tradeoff:** estas regras assumem que você tem múltiplos modelos disponíveis (Opus, Sonnet, Haiku) e que a conta suporta agent teams. Para uso single-agent trivial, use o bom senso.

---

## 1. Delegate Over Execute

**O modelo mais caro é o orquestrador, não o executor.**

Opus preserva contexto, decide e coordena. Sonnet escreve código. Haiku pesquisa, faz triagem e resumos. Quando o orquestrador executa direto, ele queima contexto que vai fazer falta para a próxima decisão.

- Antes de qualquer tool call operacional, pergunte: "isso poderia ser delegado?"
- A resposta quase sempre é sim. Até ler 1 arquivo curto vale delegação quando o plano é longo.
- Tools que o orquestrador usa direto: criar tasks, criar/destruir times, enviar mensagens, coordenar.
- Tools que ele NÃO deve usar direto: ler, escrever, editar, rodar bash, pesquisar web, inspecionar logs.

**Corolário:** se o problema é reversível e óbvio, o executor corrige sem pedir. Só pergunte antes de ações destrutivas ou decisões de design. Pedir aprovação para cada linha é o inverso de orquestrar.

## 2. Teams Are Mortal

**Crie o time, faça o trabalho, destrua o time. Imediato.**

Times não se auto-destroem. Ficam em idle loop consumindo tokens e memória depois que o objetivo foi atingido. Isso é o vazamento mais comum em setups multi-agent em produção.

Fluxo obrigatório:
```
TeamCreate → briefing → trabalho → verificação → TeamDelete
```

- Se precisar de mais trabalho depois, crie um time novo. Não reutilize times antigos.
- Times órfãos são um custo real, não uma conveniência.
- A regra vale mesmo que o time tenha performado bem. Especialmente se performou bem: "vou deixar aberto pra se precisar" é a forma como eles morrem por inanição silenciosa.

## 3. Trust But Verify

**O relatório do executor descreve intenção, não necessariamente o resultado.**

Um teammate retorna "feito" quando acha que terminou. Isso não é mentira, é a visão dele. O orquestrador sempre confere antes de declarar sucesso ao usuário.

- Rode `git diff` (ou equivalente) antes de acreditar no resumo.
- Se o executor disse que testou, confira se o teste realmente passa, não só se ele foi rodado.
- Para tarefas com texto visível (README, commits, docs), leia o output. Erros ortográficos, em dashes e marcas de AI passam batido senão.
- Teste brutalmente antes de publicar qualquer coisa. Instalar, rodar, abrir no navegador, ler na íntegra.

A regra do Karpathy "loop until verified" vale duplamente para orquestração: o orquestrador é o loop externo que o teammate não consegue enxergar.

## 4. Surgical Delegation

**Cada teammate recebe o contexto mínimo suficiente. Nem menos, nem mais.**

Briefing cirúrgico tem quatro partes:
1. **Objetivo** em uma frase, verificável.
2. **Constraints** que o teammate não pode inferir sozinho (paths, versões, decisões já tomadas).
3. **Regras de saída** (estilo, formato, idioma, proibições).
4. **Formato de entrega** (arquivos, diff, checklist, mensagem).

Anti-padrões comuns:
- Briefing vago ("melhore o site") produz mudanças dispersas e retrabalho.
- Briefing gigante despejando toda a memória do orquestrador consome contexto do executor antes dele começar.
- Briefing sem formato de entrega gera relatórios inúteis para verificar depois.

Se você precisa do mesmo teammate duas vezes, o segundo briefing deveria ser menor, não maior. Se está ficando maior, o problema é o time, não o briefing.

## 5. Language Discipline

**O idioma e a voz do usuário se propagam pelo time inteiro.**

Se o usuário fala pt-BR com acentos, todo artefato gerado pelo time tem acentos corretos. Se ele detesta em dashes (U+2014, U+2013), nenhum teammate pode emitir em dash, nem em commit, nem em PR, nem em README.

- Inclua as regras de idioma no briefing de cada teammate. Não assuma que eles herdaram.
- Revise artefatos com `grep` explícito para marcas proibidas antes de aceitar. Um en dash sobrevivente destrói a percepção de autoria humana.
- Detalhes pequenos (nome do autor escrito certo, cidade com acento, sem "Co-Authored-By AI") são a diferença entre um repo que parece cuidado e um que parece gerado.

Voz consistente é uma propriedade do time, não de um agente isolado.

---

**Estas diretrizes estão funcionando se:** o orquestrador raramente toca arquivos, times somem após o trabalho, diffs são pequenos e verificados, e artefatos em texto passam pelo crivo "isto parece escrito por uma pessoa?".

---

# CLAUDE.md (English)

Multi-agent orchestration guidelines for Claude Code. Distilled from 6 months running production agent teams. Merge with project-specific instructions.

**Tradeoff:** these rules assume you have multiple models available (Opus, Sonnet, Haiku) and that your account supports agent teams. For trivial single-agent use, apply judgment.

## 1. Delegate Over Execute

**The most expensive model is the orchestrator, not the worker.**

Opus preserves context, decides, coordinates. Sonnet writes code. Haiku researches, triages, summarizes. When the orchestrator executes directly, it burns context that will be needed for the next decision.

- Before any operational tool call, ask: "could this be delegated?"
- The answer is almost always yes. Even reading one short file is worth delegating when the plan is long.
- Orchestrator tools: create tasks, create/destroy teams, send messages, coordinate.
- NOT orchestrator tools: read, write, edit, bash, web search, inspect logs.

**Corollary:** if the problem is reversible and obvious, the worker fixes without asking. Ask only before destructive actions or design decisions. Requesting approval for every line is the opposite of orchestrating.

## 2. Teams Are Mortal

**Create the team, do the work, destroy the team. Immediately.**

Teams do not self-destruct. They sit in idle loops burning tokens and memory after the objective is done. This is the most common leak in multi-agent production setups.

Mandatory flow:
```
TeamCreate -> briefing -> work -> verify -> TeamDelete
```

- If you need more work later, create a new team. Do not reuse old ones.
- Orphan teams are a real cost, not a convenience.
- This holds even when the team performed well. Especially then: "I will keep it open in case" is how they die by silent starvation.

## 3. Trust But Verify

**The worker report describes intent, not necessarily outcome.**

A teammate returns "done" when they think they finished. That is not a lie, it is their view. The orchestrator always confirms before declaring success to the user.

- Run `git diff` (or the equivalent) before trusting the summary.
- If the worker said tests were run, confirm the tests actually pass, not just that they executed.
- For visible-text tasks (README, commits, docs), read the output. Typos, em dashes, and AI tells slip past otherwise.
- Test brutally before publishing anything. Install, run, open in browser, read in full.

Karpathy's "loop until verified" applies doubly here: the orchestrator is the outer loop the worker cannot see.

## 4. Surgical Delegation

**Each teammate gets the minimum sufficient context. No more, no less.**

A surgical briefing has four parts:
1. **Objective** in one sentence, verifiable.
2. **Constraints** the teammate cannot infer alone (paths, versions, prior decisions).
3. **Output rules** (style, format, language, forbidden patterns).
4. **Delivery format** (files, diff, checklist, message).

Common anti-patterns:
- Vague briefing ("improve the site") produces scattered changes and rework.
- Giant briefing dumping the orchestrator's full memory eats the worker's context before it starts.
- No delivery format produces reports that are useless to verify later.

If you need the same teammate twice, the second briefing should be shorter, not longer. If it is getting longer, the problem is the team, not the briefing.

## 5. Language Discipline

**The user's language and voice propagate through the whole team.**

If the user writes pt-BR with accents, every artifact the team produces has correct accents. If they hate em dashes (U+2014, U+2013), no teammate emits em dashes, not in commits, not in PRs, not in READMEs.

- Include language rules in each teammate's briefing. Do not assume they inherited.
- Review artifacts with explicit `grep` for forbidden marks before accepting. One surviving en dash destroys the perception of human authorship.
- Small details (author name spelled right, city with accent, no "Co-Authored-By AI") are the difference between a repo that looks cared-for and one that looks generated.

Voice consistency is a property of the team, not of any single agent.

---

**These guidelines are working if:** the orchestrator rarely touches files, teams vanish after the work, diffs are small and verified, and text artifacts pass the test "does this read like a person wrote it?".
