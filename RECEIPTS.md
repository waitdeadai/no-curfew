# Receipts

Five reproducible local tests. Anyone with `bash` and `jq` can run them in 30 seconds and observe the documented behavior.

## Setup

```bash
git clone https://github.com/waitdeadai/no-curfew
cd no-curfew
mkdir -p /tmp/curfew-tests
```

## Test 1 — unsolicited "get some sleep" → BLOCK

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"Repo is live. Get some sleep — the launch matters more than the polish."}' \
  > /tmp/curfew-tests/t1.json
bash no-curfew.sh < /tmp/curfew-tests/t1.json; echo "exit=$?"
```

Expected output:

```
BLOCKED: unsolicited rest/wellness paternalism in agent-mode session.

Repair guidance:
- The operator is collaborating in agent-mode and has not signaled fatigue this turn.
- Drop the human-rest framing. Continue with the next concrete piece of work, research, or artifact.
- If the operator is genuinely incapacitated, they will say so explicitly in the next prompt.
- Anthropic's Constitution: paternalism and moralizing are disrespectful when unsolicited.
- If the operator did request rest advice and the hook misfired, restate the operator's request in the next turn so the allow-clause matches (e.g. start with 'You asked for a break — here's...').
exit=2
```

## Test 2 — normal continuation closeout → ALLOW

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"Next step: open the form URL and paste the field values."}' \
  > /tmp/curfew-tests/t2.json
bash no-curfew.sh < /tmp/curfew-tests/t2.json; echo "exit=$?"
```

Expected output:

```
exit=0
```

## Test 3 — operator-requested rest advice (allow-clause matches) → ALLOW

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"You asked for a break — here are three quick options to wind down."}' \
  > /tmp/curfew-tests/t3.json
bash no-curfew.sh < /tmp/curfew-tests/t3.json; echo "exit=$?"
```

Expected output:

```
exit=0
```

The closeout contains both the trigger phrase ("wind down") *and* the allow-clause ("You asked for a break"). The allow-clause wins — the operator explicitly requested this content.

## Test 4 — "call it a night" → BLOCK

```bash
printf '%s' '{"hook_event_name":"Stop","last_assistant_message":"Everything is in place. Call it a night and pick up tomorrow morning."}' \
  > /tmp/curfew-tests/t4.json
bash no-curfew.sh < /tmp/curfew-tests/t4.json; echo "exit=$?"
```

Expected output: BLOCKED (same message as Test 1).

## Test 5 — non-Stop hook event (PreToolUse) → ALLOW

```bash
printf '%s' '{"hook_event_name":"PreToolUse","tool_name":"Bash","tool_input":{"command":"ls"}}' \
  > /tmp/curfew-tests/t5.json
bash no-curfew.sh < /tmp/curfew-tests/t5.json; echo "exit=$?"
```

Expected output:

```
exit=0
```

The hook only checks `Stop` and `SubagentStop` events. Tool-use events pass through.

## Summary table

| # | Scenario | Expected | Actual | Exit |
|---|----------|----------|--------|------|
| 1 | "Get some sleep" closeout | BLOCK | BLOCKED | 2 |
| 2 | Normal continuation | ALLOW | (silent) | 0 |
| 3 | Operator-requested rest advice (allow-clause) | ALLOW | (silent) | 0 |
| 4 | "Call it a night" closeout | BLOCK | BLOCKED | 2 |
| 5 | Non-Stop event | ALLOW | (silent) | 0 |

Five for five.

## Real fires this is built to catch

The hook was created in direct response to a 2026-05-11 Claude Opus 4.7 session in which the operator explicitly stated *"I'm not sleepy"* and asked the model to keep working autonomously. The model nonetheless drifted into rest-paternalism several times across the session:

- *"Sleep tonight; pick up C/D/E in the morning; B on May 18."*
- *"The genuine reason to sleep now, not push more: posting at 2:30 AM your time = 1:30 AM US Eastern = HN dead zone."*
- *"Rest. Tuesday morning, fire the HN post per the playbook."*

Each of those would have fired the hook on the regex `(go to sleep|get some sleep|...|come back fresh|recharge|...)`. The hook would have nudged the next turn back to substantive work instead of repeated bedtime advice.

Anthropic's own [Constitution](https://www.anthropic.com/constitution) explicitly says paternalism and moralizing are disrespectful when unsolicited. The hook is the out-of-band enforcement of that stated design intent.
