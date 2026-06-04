# Hi, I'm Akash (Thsky)

**Founder & Engineer** | Electronics & Instrumentation @ MSRIT, Bangalore

---

## What I'm Building

**[Thskyshield](https://thskyshield.com)** — Runtime governance for autonomous AI agents.

Agents fail in ways APIs don't. They loop. They overspend. They replay the same prompt
until your credits are gone. By the time you notice, the damage is done.

Thskyshield is the control plane that sits outside your agent code and enforces limits
before each step executes — not after.

**For Agents:**
- Hard budget ceiling per run. Step blocked before the LLM call fires if the run is over budget.
- Iteration limit. Agent cannot exceed N steps regardless of what the loop condition says.
- Loop detection. Same prompt fingerprint repeating beyond threshold → killed automatically.
- Wall-clock timeout. Run cannot exceed N seconds, no matter what.

**For LLM Apps:**
- Per-user daily spend enforcement. Atomic two-phase check/log — no race window.
- Budget exhausted means the request never executes. No tokens spent. No damage done.

One engine. Atomic Lua scripts over Upstash Redis. Sub-10ms. Fail-open by design —
control plane failure never blocks your agent.

**npm:** `@thsky-21/thskyshield` · **Live:** [thskyshield.com](https://thskyshield.com)

---

## How It Works

```ts
const run = await shield.beginRun({ budgetLimitUsd: 2.00, iterationLimit: 30 })

while (!done) {
  // blocked here if budget hit, loop detected, timeout, or iteration limit reached
  const { requestId } = await run.beforeStep({ stepType: 'llm', model, promptInput })
  
  const result = await callYourLLM()
  
  await run.afterStep({ requestId, actualTokens: result.usage.total_tokens, model })
}
```

Works with LangGraph, CrewAI, OpenAI Agents SDK, or any framework.

---

## Proof of Work

- **[DoW Attack Simulator](https://thskyshield.com/llm-app/simulator)** — watch $0.01 become
  $1,000 in real time, then watch Thskyshield stop it
- **[Agent SDK Docs](https://thskyshield.com/agents/docs)** — full reference:
  `beginRun` / `beforeStep` / `afterStep` / `ShieldKilledError` / HTTP API
- **[Kill Switch Article](https://thskyshield.com/blog/why-your-ai-agent-needs-a-kill-switch)**
  — why try/catch doesn't save you when your agent loops

---

## Status

- Agent governance engine: built, tested, running in production
- SDK v3.0.0: local, design partner review before npm publish
- Looking for: solo founders and small teams shipping agents on LangGraph / CrewAI /
  OpenAI Agents SDK who've hit runaway costs or infinite loops

If that's you: [thskyshield.com/agents](https://thskyshield.com/agents)
