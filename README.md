# ttfx-zoo — five frontier models port ttfx with retrieval instead of context

Split out of [omacom-io/ttfx](https://github.com/omacom-io/ttfx) at DHH's
request, so benchmark experiments don't sit in the production PR queue.

Each model was given **DHH's exact challenge** — same `plan.md`, same Python
reference — with **one** change:

> the 396k-token reference was **retrieved per ask** through a holographic
> memory layer instead of being re-sent in context.

Everything else is his harness. Every effect is gated on **compiling AND
rendering styled ANSI animation** — not on a model claiming success.

The upstream project README is preserved here as
[`UPSTREAM_README.md`](UPSTREAM_README.md).

---

## Results

| model | effects | cost (provider-billed) | DHH paid | delta |
|---|---|---|---|---|
| `deepseek/deepseek-v4-pro-0813` | 37/37 | **$4.4898** | $23 | −80.5% |
| `x-ai/grok-4.6` | 37/37 | **$10.4965** | $55 | −80.9% |
| `anthropic/claude-sonnet-5` | 37/37 | **$17.2118** | — | not run by DHH |
| `openai/gpt-5.6-sol-pro` | 37/37 | **$30.2842** | $43 | −29.6% |
| `anthropic/claude-fable-5` | 37/37 | **$54.9769** | $550 | −90.0% |

Costs are **provider-billed**, not estimated — every ask is a line in that
model's `LEDGER.jsonl`. The totals above were recomputed from those files, not
copied from the original PR titles.

```
alternates/
  deepseek_deepseek-v4-pro-0813/   88 asks   LEDGER.jsonl + port/
  x-ai_grok-4_6/                   81 asks
  anthropic_claude-sonnet-5/       96 asks
  openai_gpt-5_6-sol-pro/          50 asks
  anthropic_claude-fable-5/        81 asks
```

Each directory holds the full Rust port plus its ledger. The five original
branches (`zoo/ds`, `zoo/grok`, `zoo/sonnet`, `zoo/sol`, `zoo/fable`) are
preserved here too, if you want them one model at a time.

---

## What is actually being claimed

**Claimed:** for a task whose reference material is far larger than the ask,
retrieving the relevant slice per call costs materially less than re-sending
the whole reference every call — while still passing the same gate.

**Not claimed:** that these models beat DHH's runs on quality, that the ports
are equivalent to his, or that retrieval helps on every workload. It does not.
When the reference is small, retrieval costs *more* than simply sending it —
the advantage only appears once the corpus is much larger than the question.

---

## Honest caveats

Read these before quoting a number.

- **Ask counts differ per model** (50–96 for the same 37 effects) because
  failures are retried. A model that needed more attempts spent more; the cost
  column already includes every retry.
- **`x-ai/grok-4.6` was unservable through our OpenRouter key** for part of
  this work — the provider rejected the upstream key, and payment still
  settled on the resulting error responses. That waste is in its ledger. The
  37/37 stands; the cost is, if anything, pessimistic.
- **The render gate is strict, and models fail it in ways that look like
  success.** Several effects compiled, ran, and emitted nothing. The cause was
  never styling — it was invented crate APIs, and a name mismatch between an
  effect's `fn name()` and the harness's invocation. Those retries are in the
  ledgers.
- **DHH's baselines are his reported figures**, not re-measured by us. The
  comparison is only as good as those numbers.
- `claude-sonnet-5` has no baseline because DHH did not run it. It is included
  for shape, not for a delta.

---

## Reproducing

The retrieval layer is [openzoo](https://openzoo.fun) — pay-per-call model
access with a holographic memory layer in front; no API key, no account.

```bash
npx openzoo                      # local OpenAI-compatible proxy on :8402
npx openzoo bind <dir-or-file>   # bind a corpus, get a context id
```

Then point any OpenAI-compatible harness at `http://localhost:8402/v1` and send
`X-HRR-Context: <id>` with a small body instead of the whole corpus.

---

Original PRs, for the discussion history:
[#13](https://github.com/omacom-io/ttfx/pull/13) ·
[#14](https://github.com/omacom-io/ttfx/pull/14) ·
[#15](https://github.com/omacom-io/ttfx/pull/15) ·
[#16](https://github.com/omacom-io/ttfx/pull/16) ·
[#17](https://github.com/omacom-io/ttfx/pull/17)

Upstream ttfx is © its authors and carries their license; this repo adds only
the `alternates/` tree.
