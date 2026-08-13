# Hi, I'm Akash

**Founder & Engineer** | Electronics & Instrumentation @ MSRIT, Bangalore

---

## What I'm Building

**[Thskyshield](https://thskyshield.com)** — runtime governance for autonomous agents.

> **Your agent can only do what you said it could.**

Every run declares a boundary: how much it may spend, how many steps it may take,
how long it may run, which tools it may call, and whether it may write. The check
happens **before the call fires**, in one atomic round-trip, from a control plane
that is not the agent process it governs.

An agent that goes wrong doesn't crash. It retries a near-identical prompt, calls
a tool nobody sanctioned, or deletes something on the way to a reasonable goal.
`try/catch` never fires — nothing threw. Nothing in your stack has the authority
to stop a step *before* it happens. You find out from the invoice, or from the
missing rows.

### Six gates, two classes

**Containment** — refuses an action that hadn't happened yet. A kill here destroys nothing.

| Reason | Trips when |
|---|---|
| `killed_scope_tool` | the step calls a tool not in the run's `allowedTools` |
| `killed_scope_mutation` | the step's `mutationClass` exceeds the run's ceiling |
| `killed_loop` | the same prompt fingerprint repeats past `loopThreshold` |

**Resource** — exhaustion. A kill here can end work that might have succeeded, so it
takes a second explicit opt-in beyond enabling enforcement.

| Reason | Trips when |
|---|---|
| `killed_budget` | settled + reserved spend would exceed `budgetLimitUsd` |
| `killed_iterations` | the step count would exceed `iterationLimit` |
| `killed_timeout` | wall clock exceeds `timeoutSeconds` |

All three containment gates are evaluated before all three resource gates, so a run
that trips two at once reports the categorical reason, not the exhaustion one.

### Observe, then enforce

New projects start in **observe** — every gate evaluates on every step and records
what it *would* have done, and nothing is ever stopped. The dashboard makes one
checkable claim: *in the last 30 days, N runs would have been stopped, and they went
on to spend ≥$X after the point they should have ended.* Actuals only, never an
estimate — the headline is the sum of a column you can see and add up yourself.

**Enforce** is the same round-trip. It just actually kills the run.

---

## The Integration

```ts
const run = await shield.beginRun({
  budgetLimitUsd:   2.00,
  iterationLimit:   30,
  loopThreshold:    5,
  allowedTools:     ['search', 'read_file'],  // the only tools this run may call
  maxMutationClass: 'reversible',             // may write, may not destroy
})

while (!done) {
  // THE GATE — allowed, or killed, before your call fires
  const { requestId } = await run.beforeStep({
    stepType: 'tool', toolName: 'search', mutationClass: 'read_only', promptInput,
  })

  const result = await callYourToolOrModel()

  await run.afterStep({ requestId, actualTokens: result.usage, model })
}

await run.end()
```

Three calls. Works with LangGraph, CrewAI, the OpenAI Agents SDK, or a loop you
rolled yourself. Nothing to host, no agent to run.

**Fails open, never silently.** If the control plane can't be reached, the step is
allowed — and says so, in a `degradedReason` field you can count. `allowed: true`
with a reason is a fail-open; `allowed: true` with `null` is a governed allow. There
is no third shape. If you can't tell a customer how many of their steps ran
ungoverned, the record isn't an audit log.

**Your prompts never leave your process.** They're hashed locally; only a SHA-256
fingerprint crosses the wire.

---

## Check It Yourself

- **See it in 30 seconds, zero signup** — a stuck agent killed on budget in ~3.5s,
  in your own terminal. No account, no API key, no network call.
  ```bash
  npx @thsky-21/thskyshield-demo
  ```
- **[SDK source](https://github.com/Thsky-21/thskyshield-sdk)** — Apache-2.0, zero
  runtime dependencies, mirrored one-way from the control-plane repo so the source
  you read is the source that shipped. `grep -n "JSON.stringify" index.ts` shows
  you every request body it builds.
- **[What leaves your process](https://thskyshield.com/data)** — the complete
  field-by-field list, leaves / never-leaves, pinned by a test against the SDK.
- **[Run-boundary self-audit](https://thskyshield.com/audit)** — 15 questions about
  your own agent, answered in your browser. Nothing transmitted, no email field.
- **[Agent SDK docs](https://thskyshield.com/agents/docs)** —
  `beginRun` / `beforeStep` / `afterStep` / `ShieldKilledError` / HTTP API.
- **[Why your AI agent needs a kill switch](https://thskyshield.com/blog/why-your-ai-agent-needs-a-kill-switch)**
  — why `try/catch` doesn't save you.

**npm:** [`@thsky-21/thskyshield`](https://www.npmjs.com/package/@thsky-21/thskyshield) · **Live:** [thskyshield.com](https://thskyshield.com)

---

## Status

- Governance engine built, tested, running in production
- SDK live on npm, open source under Apache-2.0
- Observe mode is free
- Looking for: solo founders and small teams shipping agents on LangGraph / CrewAI /
  OpenAI Agents SDK who've hit runaway costs, infinite loops, or a tool call they
  never sanctioned

If that's you: **[thskyshield.com/agents](https://thskyshield.com/agents)**
