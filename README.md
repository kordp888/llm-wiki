# LLM Wiki

> **A 24/7 knowledge production system — local sLLMs and agents that continuously collect, distill, connect, and reuse knowledge. Nothing ships without a human gate.**

⚙️ **Status:** running 24/7 on local infrastructure — 500+ tests, ~5s suite
🇰🇷 한국어 요약은 [README.ko.md](./README.ko.md)를 참고하세요.

---

## Why

Most personal knowledge bases are graveyards: you save things, and they die there.

I wanted the opposite — a system where knowledge is **produced, not just stored**. Something that works while I sleep: collecting new information, analyzing it, structuring it, and connecting it back to everything I already know, so that every question I ask tomorrow starts from a richer base than today.

And I wanted it to run **locally**. Not every thought needs to go through a cloud API — small local LLMs (sLLMs) are now good enough to summarize, classify, and fact-check on your own machine, cutting cost and external dependency to near zero.

## The One-Sentence Design

> News gets collected → a human picks on a phone → only picked items become research outputs → **nothing publishes automatically.**

The governing principle across the whole pipeline: **model output approves nothing.** Facts, alerts, handoffs, renders, and publication all pass through a human gate.

## A Day in the Pipeline

```
07:00  collect      gather → dedupe → channel-fit scoring (local sLLM) → review file
07:03  propose      rank entity candidates → inject approval rows into review file
07:05  publish      review file → mobile vault (one-way, rendered as tappable checklist)
08:00  check-in     health verdict + today's queue → Telegram

       … human checks items on a phone during the day …

21:25  readback     mobile checks → wiki  (additive-only: [ ]→[x], nothing else)
21:27  apply        checked entities → approved registry
21:28  events       approved entities → event binding + cloud-context pass
21:30  check-in     evening health verdict (includes event freshness) → Telegram
21:35  research     checked news → Fact Packs · Content Briefs · queue
```

**Ordering is a contract.** Steps that read another step's output are scheduled after it — and that dependency is documented, tested, and was learned the hard way.

## Intelligence Layers

Cost policy dictates placement. Metered APIs are **disabled by default** and cannot be called without an explicit flag.

| Layer | Means | Used for |
|---|---|---|
| 1 · Deterministic | hashes, regex, scoring | dedupe · channel scoring · contradiction detection |
| 2 · Local sLLM (Ollama) | Qwen · Gemma · EXAONE · DeepSeek | fit judgment · drafts · agent conversation |
| 3 · Subscription cloud | 1M-context models | daily wide-context synthesis pass |
| 4 · Metered API | flag-gated | nothing, unless explicitly enabled per run |

Local models cap at 64K context, and the system prompt eats ~60% of it — so **long-input synthesis is a layer-3 job by design**, not an accident.

## Safety Architecture — every rule came from a real incident

- **Additive-only sync.** Mobile checks propagate `[ ]`→`[x]` **only**. Unchecks, deletions, and row removals never flow back. A bidirectional mirror once propagated a mobile-side deletion and wiped 171 wiki files — the fix wasn't a better merge, it was **making destructive signals unrepresentable**. Rows are matched by stable markers, not line positions, because mobile editors reorder files just by opening them.
- **Three human gates.** Entity approval (phone check → registry), engine handoff (separate approval file), data-engine activation (10-gate checklist). Without the gate, the pipeline structurally produces zero bindings — it doesn't "warn," it can't proceed.
- **Prompt-injection defense skeleton.** Every prompt touching external input uses the same 4-part frame: trust boundary declaration → instruction-execution ban → fixed output schema → quarantined data last. And prompts are never trusted alone — return values are code-validated (key sets, types, allowed values; model-cited IDs are accepted only if they exist in the input).
- **Artifacts over exit codes.** Three cron jobs once finished `ok` while the pipeline bound nothing. **`exit 0` does not mean success** — health verdicts judge output artifacts (counts, freshness), never process status.

## Stack

| Layer | Technology |
|---|---|
| Local models | Qwen, Gemma, EXAONE, DeepSeek — compared & selected per task |
| Cloud models (selective) | large-context models for synthesis; metered APIs flag-gated |
| Agent | Hermes Agent (scheduled orchestration + Telegram interface) |
| Storage | Obsidian-compatible Markdown + JSON, git-versioned, local-only |
| Verification | 500+ unit tests over pipeline scripts, ~5s full run |

## Wiki Concept

Each entry carries structured frontmatter (source, captured time, type, review status, tags), which makes the wiki:

1. **Searchable** by topic, time, and provenance
2. **Trustworthy** — reviewed vs. unreviewed knowledge is explicit
3. **Reusable** — agents can query it as context for new work

## Future

- [ ] Public template version others can self-host
- [ ] Richer cross-linking / graph view
- [ ] Quality benchmarks: sLLM vs cloud LLM on summarization & fact-check tasks

---

**Note:** This repository documents the architecture and workflow. The wiki contents, credentials, and personal data are not included.

*Built by [Derrick Hwang](https://github.com/kordp888) — AI Native Product Manager. Why comes first. Building comes next.*
