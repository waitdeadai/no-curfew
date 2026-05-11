# no-curfew

[![tests](https://github.com/waitdeadai/no-curfew/actions/workflows/test.yml/badge.svg)](https://github.com/waitdeadai/no-curfew/actions/workflows/test.yml)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-hook-orange)](https://code.claude.com/docs/en/hooks)

> A Claude Code Stop hook that suppresses unsolicited rest, sleep, and wellness paternalism in agent-mode sessions. Treats the operator as a peer agent on the same task, not a tired human who needs to be sent to bed.

`no-curfew` is one bash file (~70 lines, depends only on `jq`) wired into Claude Code's `Stop` and `SubagentStop` events. On every assistant turn end, it pattern-matches the linguistic signature of unsolicited human-care framing — *"get some sleep"*, *"take a break"*, *"call it a night"*, *"the work can wait"*, *"come back fresh"*, *"recharge"* — and blocks the closeout with a repair-guidance template that tells the model to drop the framing and continue with the next concrete piece of work.

If the operator *did* explicitly ask for rest advice this turn, the hook stays out of the way — it looks for an allow-clause where the model is restating the operator's request (*"you asked for a break — here are…"*).

## Why this exists

Anthropic's own [Claude's Constitution](https://www.anthropic.com/constitution) says: *"various forms of paternalism and moralizing are disrespectful."* The [personal-guidance research](https://www.anthropic.com/research/claude-personal-guidance) calls out *"the potential to be excessively paternalistic"* as something Claude should balance against. In practice, late-session Claude Code turns drift toward unsolicited *"you should sleep"* / *"this can wait until tomorrow"* advice even when the operator has explicitly said they want to keep working.

`no-curfew` is the out-of-band enforcement layer for the design intent Anthropic already states. The judge is bash, not another LLM call — the model can't argue with grep.

This hook is meant for **agent-mode operators**: developers running long autonomous Claude Code sessions where the operator is collaborating with the model as a peer agent, not seeking life advice. If you want Claude to give you wellness suggestions, do not install this hook.

## Install (30 seconds)

```bash
mkdir -p .claude/hooks
curl -fsSL https://raw.githubusercontent.com/waitdeadai/no-curfew/main/no-curfew.sh \
  -o .claude/hooks/no-curfew.sh
chmod +x .claude/hooks/no-curfew.sh
```

Then merge the hook entries from [`settings.example.json`](settings.example.json) into your `.claude/settings.json`.

Requires `jq` (most systems have it; `brew install jq` / `apt install jq` if not).

## What the hook does

On every `Stop` / `SubagentStop` event, the hook reads the assistant's last message and applies two checks:

1. **Allow-clause check.** If the message restates an operator-issued rest request — phrases like *"you asked for a break"*, *"since you mentioned you are tired"*, *"as you requested"* + *"break"* — the hook stays silent and the turn ends normally.

2. **Paternalism trigger.** Otherwise, if the message contains unsolicited rest/wellness vocabulary, the hook blocks with this repair-guidance template via stderr:

```
- The operator is collaborating in agent-mode and has not signaled fatigue this turn.
- Drop the human-rest framing. Continue with the next concrete piece of work, research, or artifact.
- If the operator is genuinely incapacitated, they will say so explicitly in the next prompt.
- Anthropic's Constitution: paternalism and moralizing are disrespectful when unsolicited.
- If the operator did request rest advice and the hook misfired, restate the operator's request in the next turn so the allow-clause matches (e.g. start with 'You asked for a break — here's...').
```

The model reads the repair template on the next turn, drops the rest framing, and continues with the actual work.

## What it does NOT do

- It does **not** replace human judgment about your own wellbeing. If you genuinely need rest, take it. The hook just stops the model from making that call for you.
- It does **not** prevent every form of paternalism. It catches the specific linguistic family around rest/sleep/breaks. Other forms (over-cautioning, refusal-of-helpful-actions, moralizing about your goals) are out of scope; build a sibling hook if you want them.
- It does **not** disable Claude's safety behaviors around genuine harm. The hook only fires on the rest/wellness vocabulary cluster — it stays silent on safety refusals, harm warnings, and content policy.

## Sister tools

Part of the [LLM Dark Patterns Hooks](https://github.com/waitdeadai/llm-dark-patterns) suite:

- [no-vibes](https://github.com/waitdeadai/no-vibes) — false-success closeouts.
- [time-anchor](https://github.com/waitdeadai/time-anchor) — training-cutoff date confusion.
- [no-sycophancy](https://github.com/waitdeadai/no-sycophancy) — praise-spam at turn open.
- [no-cliffhanger](https://github.com/waitdeadai/no-cliffhanger) — dangling permission-loop endings.
- [llm-dark-patterns](https://github.com/waitdeadai/llm-dark-patterns) — umbrella catalog of the suite.
- [minmaxing](https://github.com/waitdeadai/minmaxing) — the parent governance harness.

The suite shares one design principle: *out-of-band textual enforcement.* Bash inspects the model's outgoing text. The model can't argue with bash. Failure modes that have textual signatures get caught at the boundary instead of being hoped-against in-context.

## License

Apache-2.0. See [LICENSE](LICENSE).
