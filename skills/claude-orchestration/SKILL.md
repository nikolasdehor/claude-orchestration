---
name: claude-orchestration
description: Multi-agent orchestration guidelines for Claude Code. Use when running agent teams with multiple models (Opus orchestrating, Sonnet/Haiku executing). Covers when to delegate vs execute, team lifecycle, briefing shape, verification, and preserving user voice across teammates.
license: MIT
---

# claude-orchestration

Multi-agent orchestration guidelines distilled from running production agent teams (OpenClaw/Devinho, custom swarms, persistent tmux agent teams). Complements single-agent cognitive guidelines like forrestchang/andrej-karpathy-skills.

**Tradeoff:** assumes multiple models available (Opus, Sonnet, Haiku) and that agent teams are supported. For trivial single-agent use, apply judgment.

## 1. Delegate Over Execute

The most expensive model is the orchestrator, not the worker. Opus preserves context, decides, coordinates. Sonnet writes code. Haiku researches, triages, summarizes. When the orchestrator executes directly, it burns context that the next decision will need.

- Before any operational tool call, ask: "could this be delegated?"
- The answer is almost always yes. Even reading one short file is worth delegating when the plan is long.
- Orchestrator tools: create tasks, create/destroy teams, send messages, coordinate.
- NOT orchestrator tools: read, write, edit, bash, web search, inspect logs.

Corollary: if the problem is reversible and obvious, the worker fixes without asking. Ask only before destructive actions or design decisions.

## 2. Teams Are Mortal

Teams do not self-destruct. They sit in idle loops burning tokens and memory after the objective is done. This is the most common leak in multi-agent production setups.

Mandatory flow:

```
TeamCreate -> briefing -> work -> verify -> TeamDelete
```

- If you need more work later, create a new team. Do not reuse old ones.
- Orphan teams are a real cost, not a convenience.
- This holds especially when the team performed well. "I will keep it open in case" is how they die by silent starvation.

## 3. Trust But Verify

The worker report describes intent, not outcome. The orchestrator always confirms before declaring success to the user.

- Run `git diff` (or equivalent) before trusting the summary.
- If the worker said tests were run, confirm they actually pass.
- For visible-text tasks (README, commits, docs), read the output. Typos and AI tells slip past otherwise.
- Test brutally before publishing anything.

## 4. Surgical Delegation

Each teammate gets the minimum sufficient context. Briefing shape:

1. **Objective** in one sentence, verifiable.
2. **Constraints** the teammate cannot infer (paths, versions, prior decisions).
3. **Output rules** (style, format, language, forbidden patterns).
4. **Delivery format** (files, diff, checklist, message).

If you need the same teammate twice, the second briefing should be shorter. If it grows, the team is the problem, not the briefing.

## 5. Language Discipline

User language and voice propagate through the whole team.

- Include language rules in each teammate's briefing. Do not assume inheritance.
- Review artifacts with explicit `grep` for forbidden marks (em dashes, AI tells) before accepting.
- Small details (author name spelling, city accents, no "Co-Authored-By AI") are the difference between a repo that looks cared-for and one that looks generated.

## Working check

These guidelines are working if: orchestrator rarely touches files, teams vanish after work, diffs are small and verified, and text artifacts pass the test "does this read like a person wrote it?".

See full guidelines and examples at https://github.com/nikolasdehor/claude-orchestration.
