# claude-orchestration

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

Um único `CLAUDE.md` com 5 princípios para orquestrar times de agentes no Claude Code sem desperdiçar contexto nem deixar times órfãos rodando.

Pt-BR | [English](#claude-orchestration-english)

## Por que mais um CLAUDE.md

O excelente [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) cobriu os **princípios cognitivos** do agente individual: pense antes de codar, simplifique, mude cirurgicamente, siga critérios verificáveis.

Este repo cobre a camada seguinte: **como orquestrar múltiplos agentes em produção**. Destilado de 6 meses rodando swarms customizados (OpenClaw/Devinho, 11 agentes paralelos, agent teams persistentes em tmux) e de iterações reais sobre o que funciona e o que vaza recursos.

Os dois são complementares. O do Karpathy diz ao agente como se comportar. Este diz ao orquestrador como comandar o time.

## Os 5 princípios

| Princípio | Endereça |
|-----------|----------|
| **Delegate Over Execute** | Orquestrador queimando contexto com trabalho de executor |
| **Teams Are Mortal** | Times órfãos em idle loop após o trabalho terminar |
| **Trust But Verify** | "Feito" do teammate diferente de "verificado" pelo orquestrador |
| **Surgical Delegation** | Briefings vagos ou gigantes que produzem retrabalho |
| **Language Discipline** | Voz do usuário se perdendo quando o time cresce |

Leia tudo em [CLAUDE.md](./CLAUDE.md). Exemplos antes/depois em [EXAMPLES.md](./EXAMPLES.md).

## Quick install

**Opção A: Plugin (recomendado)**

Dentro do Claude Code, adicione o marketplace:
```
/plugin marketplace add nikolasdehor/claude-orchestration
```

Depois instale o plugin:
```
/plugin install claude-orchestration@claude-orchestration
```

**Opção B: CLAUDE.md direto no projeto**

Novo projeto:
```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/nikolasdehor/claude-orchestration/main/CLAUDE.md
```

Projeto existente, anexando:
```bash
printf "\n\n" >> CLAUDE.md
curl https://raw.githubusercontent.com/nikolasdehor/claude-orchestration/main/CLAUDE.md >> CLAUDE.md
```

**Opção C: copiar e colar**

Abra [CLAUDE.md](./CLAUDE.md), copie, cole no seu.

## Quando isto ajuda

- Você usa Claude Code com múltiplos modelos (Opus orquestrando, Sonnet/Haiku executando).
- Você já criou agent teams e notou que alguns ficam "esquecidos" rodando.
- Seus diffs têm alterações dispersas que você não pediu.
- READMEs gerados pelo time têm marcas de AI (em dashes, estrutura genérica).
- O orquestrador está consumindo contexto lendo arquivos em vez de decidir.

## Quando isto não ajuda

- Uso single-agent trivial. As regras assumem múltiplos modelos e agent teams.
- Scripts pontuais de um arquivo só. Use o [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills).

## Créditos

Estrutura e formato inspirados em [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills). Conteúdo próprio, destilado da experiência de rodar OpenClaw/Devinho e outros setups multi-agent em produção.

## Licença

[MIT](./LICENSE). Autor: Nikolas de Hor (nikolasdehor79@gmail.com), Goiânia, Brasil.

---

# claude-orchestration (English)

A single `CLAUDE.md` with 5 principles to orchestrate agent teams in Claude Code without wasting context or leaving orphan teams running.

## Why another CLAUDE.md

The excellent [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) covers the **cognitive principles** of a single agent: think before coding, simplify, change surgically, follow verifiable criteria.

This repo covers the next layer: **how to orchestrate multiple agents in production**. Distilled from 6 months running custom swarms (OpenClaw/Devinho, 11 parallel agents, persistent agent teams in tmux) and real iteration on what works and what leaks resources.

They are complementary. Karpathy's tells the agent how to behave. This one tells the orchestrator how to run the team.

## The 5 principles

| Principle | Addresses |
|-----------|-----------|
| **Delegate Over Execute** | Orchestrator burning context on worker tasks |
| **Teams Are Mortal** | Orphan teams in idle loop after the work ends |
| **Trust But Verify** | Worker "done" is not orchestrator "verified" |
| **Surgical Delegation** | Vague or bloated briefings that cause rework |
| **Language Discipline** | User's voice getting lost as the team grows |

Full text in [CLAUDE.md](./CLAUDE.md). Before/after examples in [EXAMPLES.md](./EXAMPLES.md).

## Quick install

**Option A: Plugin (recommended)**

Inside Claude Code, add the marketplace:
```
/plugin marketplace add nikolasdehor/claude-orchestration
```

Then install the plugin:
```
/plugin install claude-orchestration@claude-orchestration
```

**Option B: CLAUDE.md directly in the project**

New project:
```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/nikolasdehor/claude-orchestration/main/CLAUDE.md
```

Existing project, append:
```bash
printf "\n\n" >> CLAUDE.md
curl https://raw.githubusercontent.com/nikolasdehor/claude-orchestration/main/CLAUDE.md >> CLAUDE.md
```

**Option C: copy and paste**

Open [CLAUDE.md](./CLAUDE.md), copy, paste into yours.

## When it helps

- You use Claude Code with multiple models (Opus orchestrating, Sonnet/Haiku executing).
- You already created agent teams and noticed some are left running idle.
- Your diffs include scattered changes you did not ask for.
- Team-generated READMEs carry AI tells (em dashes, generic structure).
- The orchestrator is burning context reading files instead of deciding.

## When it does not help

- Trivial single-agent use. These rules assume multiple models and agent teams.
- One-off single-file scripts. Use [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) instead.

## Credits

Structure and format inspired by [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills). Content is original, distilled from running OpenClaw/Devinho and other multi-agent setups in production.

## License

[MIT](./LICENSE). Author: Nikolas de Hor (nikolasdehor79@gmail.com), Goiânia, Brazil.
