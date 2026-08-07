# Hi, I'm Akash 

**Founder & Engineer** | Electronics & Instrumentation @ MSRIT, Bangalore

---

## What I'm Building

**[Thskyshield](https://thskyshield.com)** — the control plane that stops an AI agent
from spending $200 on a $2 task.

An agent that gets stuck doesn't crash. It retries. It re-sends a near-identical
prompt, gets a near-identical non-answer, and bills you for every attempt.
`try/catch` never fires — nothing threw. Nothing in your stack has the authority
to stop a step *before* it fires. You find out at the invoice.

Thskyshield sits outside your agent loop and answers one question before every
step: **is this run still allowed to spend money?**

### One engine, two modes

**Observe** — free, default. Nothing is ever blocked. Every gate still evaluates
on every step, records what it *would* have done, and produces a **Waste Report**:
a dollar figure, built only from settled actuals, for money burned on retries and
loops you didn't know were happening. It's a lower bound by construction — never
inflated, checkable against your own provider invoice.

**Enforce** — the unlock. Same check, same round-trip. Now it actually kills the run.

### Four gates, one atomic Lua round-trip, sub-10ms

| Gate | Trips when |
|---|---|
| Budget | `spent + reserved + cost > budget` |
| Iterations | `iter > iteration_limit` |
| Loop | same prompt fingerprint repeats past threshold |
| Timeout | run exceeds its wall-clock limit |

Fail-open by design — a control-plane hiccup never blocks your agent.
Prompts never leave your process; only a SHA-256 fingerprint crosses the wire.

**npm:** [`@thsky-21/thskyshield`](https://www.npmjs.com/package/@thsky-21/thskyshield) · **Live:** [thskyshield.com](https://thskyshield.com)

---

## How It Works

```ts
const run = await shield.beginRun({ budgetLimitUsd: 2.00, iterationLimit: 30 })

while (!done) {
  // THE GATE — allowed, or killed, before your LLM call fires
  const { requestId } = await run.beforeStep({ stepType: 'llm', model, promptInput })

  const result = await callYourLLM()

  await run.afterStep({ requestId, actualTokens: result.usage.total_tokens, model })
}

await run.end()
```

Works with LangGraph, CrewAI, OpenAI Agents SDK, or any framework you've rolled yourself.

---

## Proof of Work

- **See it in 30 seconds, zero signup:**
  ```bash
  npx @thsky-21/thskyshield-demo
  ```
  A stuck agent gets killed on budget in ~3.5 seconds, in your own terminal. No account, no network call.
- **[Agent SDK Docs](https://thskyshield.com/agents/docs)** — full reference:
  `beginRun` / `beforeStep` / `afterStep` / `ShieldKilledError` / HTTP API
- **[Kill Switch Article](https://thskyshield.com/blog/why-your-ai-agent-needs-a-kill-switch)**
  — why `try/catch` doesn't save you when your agent loops

---

## Status

- Agent governance engine: built, tested, running in production
- SDK live on npm — `@thsky-21/thskyshield`
- Looking for: solo founders and small teams shipping agents on LangGraph / CrewAI /
  OpenAI Agents SDK who've hit runaway costs or infinite loops

If that's you: [thskyshield.com/agents](https://thskyshield.com/agents)
