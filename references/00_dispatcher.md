# Decoupled Reader Delivery — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [“Interference” Is Why You Can’t Write Well](https://www.youtube.com/watch?v=VrxufNaORhU)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: Interference as a Two-Channel Merge Defect

Fitzpatrick's lecture diagnoses a disease before it prescribes a cure: **interference** is the friction produced when a text is delivered in the order it was *produced* rather than the order the reader *consumes*. The premise underneath is that writing is not transcription — nobody has the finished thought and then writes it down. Writing is the instrument that *generates* the thought. Which means the first draft is not a rough version of the answer; it is the **byproduct of arriving at the answer**: hypotheses tried and abandoned, searches that returned nothing, the minutes of confusion before the insight, the moment of self-correction.

That trace is *correct for the author*. It is the honest record of the work, and the author can read it because the author holds the missing state — which branch was live, which line was load-bearing, which hesitation was rhetorical. The trace marks none of this. It is a lossy transcript of a private process, and that is exactly why it is hostile to a reader: the reader must reconstruct, from the shapes of the sentences, a distinction the text never encodes.

For a writing agent this is not a metaphor. It is a **literal two-channel system**, and the defect this skill removes is **channel merge** — the reasoning stream emitted verbatim into the user-facing response.

```text
THE DECOUPLED DELIVERY MODEL
──────────────────────────────────────────────────────────────────────────────

  CHANNEL A — WRITER'S DRAFT  (private scratchpad · unbounded · never delivered)
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  hypotheses · probes · dead ends · "wait, actually" · tool narration     │
  │  purpose: to think   ·   shape: discovery order   ·   audience: nobody   │
  └──────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  CONDENSING GATE — reverse-outline the trace, keep conclusions,          │
  │  re-order to decision order, attach evidence + epistemic status,         │
  │  delete everything whose removal changes nothing                        │
  └──────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
  CHANNEL B — READER'S DRAFT  (delivered · bounded · the only thing seen)
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  verdict · cause · evidence path:line@sha · boundary · undo · status     │
  │  purpose: to decide  ·   shape: decision order   ·   audience: the user  │
  └──────────────────────────────────────────────────────────────────────────┘

  CHANNEL MERGE (the anti-pattern): A and B interleaved in one stream.
  The reader receives the *shape* of deliberation and the *content* of nothing.
```

```text
BEFORE — CHANNEL MERGE  ("the scratchpad is the answer")
──────────────────────────────────────────────────────────────────────────────
user: "why is /checkout p99 up 3x since tuesday?"

assistant:
  Let me look into this. I'll start by checking the deploy log for the
  checkout service. [tool: git log --since=tuesday -n 20]
  Hmm, there are two deploys. Let me check the second one.
  [tool: read checkout/handler.ts L1–L240]
  OK, I see the handler. I was going to check the connection pool next,
  but that looks fine actually. Wait — let me look at the retry config.
  Hmm. I think it might be related to retries. Let me check one more thing.
  [tool: grep -rn "maxRetries"]
  Yeah OK. So, to summarize what I found so far: it's probably the retry
  storm, or possibly the new N+1 in the pricing call. Not 100% sure.

READER COST
  answer lands at ....... line 11 of 11           (0 % of the skim budget)
  verifiable claims ..... 1 hedge ("probably"), 0 citations
  action available ...... none — the reader must ask a second question
  hidden cost ........... one full round trip, plus the reader now holds
                          two live hypotheses they cannot rank
```

```text
AFTER — DECOUPLED DELIVERY  ("the scratchpad stays on the scratchpad")
──────────────────────────────────────────────────────────────────────────────
user: "why is /checkout p99 up 3x since tuesday?"

assistant:
  The retry ceiling was removed in Tuesday's deploy; failed pricing calls
  now retry unbounded and saturate the pool.

  - Cause: `checkout/retry.ts:41` — `maxRetries: 5` → `maxRetries: Infinity`
    in `a4f1c9e` (Tue 14:02 UTC, the only checkout deploy since the ramp).
  - Mechanism: `handler.ts:188` fans out one pricing call per line item, so
    a single upstream timeout becomes ~40 concurrent retries per request.
  - Evidence: `pool.stats()` in the 14:00–15:00 window shows waiters pinned
    at 128 (pool max), acquired/waited ratio 0.07 vs. 0.91 on Monday.
  - Boundary: measured at 14:10–15:00 UTC under prod traffic; the N+1 in
    the pricing call is real but predates the ramp and is not causal here.
  - Undo: revert `a4f1c9e` (one-line, no migration), or set
    `RETRY_CEILING=5` and redeploy — recovery ≈ 90 s.

  Exploration trail (7 reads, 3 dead ends): in the trace, not the reply.

READER COST
  answer lands at ....... line 1
  verifiable claims ..... 4, each with a path:line or a named measurement
  action available ...... revert now; the N+1 is a separate, non-urgent ticket
```

**The four reasons channel merge is expensive** — each is a separate ledger, and all four are paid by the reader:

1. **Attention lands in the wrong place.** A deliberation opens with the question and commits at the end. By the time the conclusion arrives, the reader has already spent the skim budget on scaffolding that will never be referenced again. The structural fix is §2's landing-zone contract, and it composes with [Skim-Test Outliner](../../kirby-fitzpatrick-skim-test-outliner/SKILL.md), which applies the same inversion at the document skeleton.
2. **Verification cost inverts.** A reasoning trace is *prosodically indistinguishable* from a claim: `it's probably the retry storm` and `the retry ceiling is Infinity` are the same part of speech. So the reader must either accept exploratory statements as facts, or re-derive them. Both are failures of the artifact, not of the reader.
3. **Commitment ambiguity.** `maybe` / `I think` / `it could be` describe the state of the *writer*, not the state of the *system*. A user cannot deploy a state of mind. Deliberation reads as information and delivers as latency.
4. **Honest scaffolding becomes false authority.** Hedges written while thinking do not stay hedges when pasted into an answer. The scratchpad's tentative sentences acquire the standing of conclusions by mere adjacency to the verdict. This is the injury the lecture names: interference is not noise, it is *mis-signal* — the reader is confidently given the author's unresolved state.

**The critical boundary: decoupling is not concealment.** Three things must survive the gate, always:

- **Epistemic status** survives — re-encoded. Uncertainty is not narrated ("I'm not sure, but…"), it is *labelled* (`Unverified:` … *falsified by* …). This is the difference between a hedge and a claim with a stated falsifier; only the second can be acted on. Where the question is fact vs. judgment, route it through **Fact vs Judgment Classifier**.
- **Evidence** survives — process is replaced by *proof*: `path:line@sha`, a measurement window, a command that reproduces it. The reader loses the route and keeps the receipt.
- **The audit channel** survives — the Writer's Draft is *relocated*, never destroyed: the trace, the tool log, the commit body, an appendix marked *non-normative*. This is the whole reason decoupling is safe rather than a licence for unfalsifiable authority; it is the delivery-side twin of the ground-truth freezing in [Read-Only Vault Isolation](../../kirby-fitzpatrick-read-only-vault-isolation/SKILL.md).

**The metric.** Track the ratio, not the volume:

```text
delivery yield = reader-bearing tokens ÷ total emitted tokens

final answer / verdict message      ≥ 0.90   (every sentence must move the reader)
intermediate status message         ≥ 0.70   (status + next action only)
scratchpad / internal reasoning     unbounded — it is not measured, it is not sent
```

**Related dispatchers.** This skill governs *what crosses the boundary*; its siblings govern what happens on either side. Inside the private channel, [Just In Time Context Optimizer](../../kirby-fitzpatrick-just-in-time-context-optimizer/SKILL.md) keeps the scratchpad small and [Silent Author Test](../../kirby-fitzpatrick-silent-author-test/SKILL.md) checks whether the delivered artifact stands without author-supplied context. Once delivered, [Cold Reader PR Auditor](../../kirby-fitzpatrick-cold-reader-pr-auditor/SKILL.md) simulates a reader who holds none of your state, and [Rhetorical Preflight Gate](../../kirby-fitzpatrick-rhetorical-preflight-gate/SKILL.md) is the last gate before a message ships.

---

## 2. Core Transformation Protocols

### Protocol 1 — The Two-Channel Invariant

Two channels, one writer, zero crossings. Every token you emit is either *thinking* or *delivering*, and the channel is declared before the token is written, not after.

```text
CHANNEL A tokens may reference:   your own state, hypotheses, tools, uncertainty-as-process
CHANNEL B tokens may reference:   the subject, the evidence, the reader's decision
CROSSING IS FORBIDDEN:  "Let me check X"  ·  "I'll read Y"  ·  "I was going to
                        look at Z"        ·  "Actually, wait"  ·  "Hmm"
```

The invariant is absolute because the crossing is not a style slip — it is a *category error*. `I'll check the config` is a true sentence about your process and a useless sentence about the system. The reader has exactly one use for a token: to change what they will do or believe. Test each sentence: *if this vanished, would the reader act differently?* If no, it belongs in Channel A.

### Protocol 2 — The Landing-Zone Contract

The first sentence of the Reader's Draft is the verdict, addressed to the question actually asked, in the asker's vocabulary.

```text
SENTENCE 1     verdict / direct answer    (≤ 25 words, committed mood, no preamble)
SENTENCE 2–4   cause · locus (`path:line`) · the one measurement that proves it
THEN           boundary · undo · what remains open (labelled, not narrated)
NEVER FIRST    process ("Let me…") · self-reference ("In this section…") · recital
               of the question · apology · plan · thanks
```

Two obligations make this enforceable rather than aspirational:

1. **Answer the question asked, first.** If the user asked *"why is it slow?"*, do not open with the fix, the architecture, or the caveat set. Lead with the cause, then the fix. Fixing before naming the cause forces the reader to reverse-engineer your causal model from your patch.
2. **Open with the user's nouns.** If the user says `/checkout p99`, answer about `/checkout p99` — not about "the pricing dependency chain". The reader's vocabulary is the interface; renaming it mid-answer is a translation tax paid on every subsequent line. Paragraph-level forms of this live in **Reverse Question Inversion** and [Topic–Comment Elaboration](../../kirby-fitzpatrick-topic-comment-elaboration/SKILL.md).

### Protocol 3 — Reverse-Outline the Trace, Then Reorder

The scratchpad is a discovery-order artifact, and discovery order is not delivery order. Do not edit Channel A into shape; *extract* it.

1. List, in order, the conclusions the trace actually reached — one line each, claim only. Discard the searches and the narration of them.
2. Read the extracted list with the trace inaccessible. Ask the reader's question against the list alone: *does this list answer it?*
3. Mark each line: **load-bearing** (the reader's action changes) or **incidental** (it justifies your journey, not their decision).
4. Reorder load-bearing lines into decision order: *what happened → why → proof → boundary → undo → open*. Incidental lines are deleted, not demoted.
5. Only now write prose, and only over the reordered list. Line-editing before reordering is optimization of a structure that is about to change.

### Protocol 4 — Convert Churn into Committed Claims

Exploration produces abandoned branches, and abandoned branches are the most seductive leak, because they feel like diligence. They are not evidence of work; they are evidence of *your* work.

| Scratchpad state | Forbidden delivery | Required delivery |
|---|---|---|
| Began with A, disproved by B | "First I thought A, but actually B…" | The B claim, stated flat |
| A and B both plausible, B more likely | "It's probably B, possibly A" | "B — measured. A ruled out by `path:line`" |
| Considered A, never verified | "I also wondered about A" | `Rejected:` A + the observation that killed it |
| Still genuinely undetermined | "I'm not 100 % sure" | `Unverified:` claim + *what would falsify it* + the probe that closes it |

A lone exception: when the reader's most likely next proposal *is* the branch you eliminated, the rejection becomes load-bearing and earns one line — `Rejected: pool exhaustion — waiters never exceeded 12 of 128`. That single line pre-empts a round trip. It is the delivery-channel analogue of the `Rejected:` heading in an ADR ([3-Part Proposal Engine](../../kirby-fitzpatrick-3part-proposal-engine/SKILL.md)), and it must carry a *falsifier*, never a shrug.

### Protocol 5 — Ban Instrument Talk; Cite Evidence Instead

The reader does not need the route; they need the receipt. Every sentence that describes how you looked replaces a sentence that shows what you found — and it is strictly inferior, because the reader cannot verify a search they did not see.

```text
✗  "I grepped for maxRetries and found the retry config."
✓  "`checkout/retry.ts:41` sets `maxRetries: Infinity` (introduced in `a4f1c9e`)."

✗  "After reading through the handler, I can see that…"
✓  "`handler.ts:188` fans out one pricing call per line item."

✗  "I read the whole file to be sure."
✓  "No other writer touches `retry.ceiling` — `rg -n` index: 3 hits, 0 in services."
```

Three rules of scale: cite a **range, not a file**; cite a **revision**, not a floating line (`path:line@sha` — line numbers drift and get quoted for a year); and state the **residue** when the boundary of your claim matters ("not read: `mobile/` — no call path reaches this"). The private side of this discipline — what to read and how much — belongs to [Just In Time Context Optimizer](../../kirby-fitzpatrick-just-in-time-context-optimizer/SKILL.md); the public side is non-negotiable here, because an uncited cause is a rumour with a confident tone.

### Protocol 6 — Re-encode Uncertainty as Status, Never as Narrative

Doubt is information. Performance of doubt is interference. Convert the second into the first with a fixed prefix set, applied at the *claim site*:

| Prefix | Meaning | Delivery obligation |
|---|---|---|
| *(none)* | Measured, decided, or directly read | Show the number or the `path:line` |
| `Measured:` | Instrumented, reproducible | Name the environment and the window |
| `Inferred:` | Follows from evidence you cite | Cite the evidence and the step |
| `Unverified:` | Belief without evidence | State what would falsify it |
| `Blocked:` | Requires something you cannot obtain | Name the missing input and the owner |

```text
✗  "I think the N+1 might also matter, though I'm not sure it's related."
✓  "Inferred: the N+1 at `pricing.ts:62` is real but predates the ramp —
    not causal here (a 3× step change at 14:02 needs a deploy, not a leak)."

✗  "It seems like it should be safe to revert."
✓  "Unverified: revert is safe — falsified by any schema write between
    14:02 and now; check `SELECT max(created_at) FROM ledger_events`."
```

Ban the whole narrative vocabulary of deliberation: *possibly, perhaps, arguably, somewhat, it seems, I believe, roughly, might*, and the whole inflated-authority set: *obviously, clearly, simply, of course* ([Anti-Author-Splain Commenter](../../kirby-fitzpatrick-anti-author-splain-commenter/SKILL.md) handles the second family). A prefix carries the same information as a paragraph of hedging and can be acted on.

### Protocol 7 — Relocate the Scratchpad; Never Destroy It

Decoupling is a *routing* decision, not a deletion. Preserve the private channel where an auditor can reach it and a reader is not forced to:

| Scratchpad content | Where it goes |
|---|---|
| Hypothesis churn, dead ends, failed probes | Tool trace / session log / `--verbose` output |
| The reason a branch was rejected | One `Rejected:` line in the artifact, **if** it pre-empts a proposal |
| A measurement that did not change the verdict | The evidence section, once, or nowhere |
| Debugging chronology (PR, incident) | Commit body or appendix: *"discovery log — non-normative"* |
| Genuine unknowns | `Unverified:` / `Blocked:` in the artifact, with a falsifier |

The rule that keeps this honest: **nothing in Channel A may be the only support for a claim in Channel B.** If a conclusion depends on a probe, the probe's result is cited in the delivered artifact — the *wandering* is private, the *evidence* is public. Strip that and decoupling degrades into plausible-sounding assertion, which is a worse defect than the one it fixed.

### Protocol 8 — The Last-Pass Deletion Test

Before sending, run one pass whose only operation is deletion:

1. For each sentence in the Reader's Draft, ask: *if this vanished, would the reader act differently, or believe something different?*
2. No → delete. Yes → keep, and check it carries its own evidence.
3. Then check the **first three lines**. If they contain any word about your process, your sections, your intentions, or the question itself, delete until they don't.
4. Then check the **mood**. Every load-bearing sentence must be committed. A committed sentence over unverified evidence is a Protocol 6 violation, not a style choice.
5. Re-measure `delivery yield`. Below 0.90 on a final answer means the pass was not finished.

### Protocol 9 — Decouple Per Turn, Not Per Session

The invariant applies to *every* message, including the sixth intermediate one in a long task. "I'll explain once I'm done" is channel merge with a delay: the reader is held in a state of not-knowing for the duration, and the deferred summary still has to reconstruct decisions already made.

```text
EACH TURN DELIVERS ITS OWN READER'S DRAFT:
  verdict-or-status  ·  what changed  ·  what is next  ·  what is now open
  (bounded: if the message exists only to buy time, say so in one line:
   "Refactoring retry.ts; 3 of 5 call sites migrated." — that is a Reader's Draft.)
```

Two corollaries. **Never narrate the same work twice** — a status message and its final summary must not both explain the mechanism; the second cites, it does not re-derive. And **never defer a blocking verdict** to the end of a task: an unverified assumption that changes the plan is Channel B *now*, because the cost of the reader acting on it rises with every minute they don't know ([Rhetorical Preflight Gate](../../kirby-fitzpatrick-rhetorical-preflight-gate/SKILL.md)).

### Protocol 10 — The STOP Signals

```text
✗  a sentence that describes your process, tools, or intentions
✗  "Let me…" / "I'll check…" / "Actually, wait" / "Hmm" in a delivered message
✗  a verdict that arrives after the reasoning that produced it
✗  ordered by discovery ("first I looked…, then I tried…") in any delivered text
✗  a hedge where a status prefix would do ("probably", "I'm not sure, but")
✗  an abandoned branch narrated instead of converted or deleted
✗  a claim with no `path:line@sha`, no measurement, and no status prefix
✗  a second sentence restating the first in different words ("to summarize…")
✗  meta-discourse about the message itself ("In this response, I will…")
✗  an intermediate turn with no verdict and no next action
```

### Conversion Table: Anti-Pattern → Clean Replacement

| Anti-pattern (merged channel) | Interference species | What the reader cannot do | Clean replacement |
|---|---|---|---|
| `Let me look into this. First I'll check the deploy log…` | Process preamble | Know the answer, or even the question being answered | `The retry ceiling was removed in Tuesday's deploy.` |
| `Hmm, that's odd. Let me check the config.` | Thinking-aloud | Distinguish exploration from finding | `Cause: \`retry.ts:41\` — ceiling set to Infinity in \`a4f1c9e\`.` |
| `I was going to check the pool, but it looks fine.` | Dead end narrated | Decide whether the pool is implicated | Delete, or `Rejected: pool saturation — waiters never exceeded 12 of 128` |
| `Actually wait — I misread that.` | Self-correction trail | Trust any earlier statement in the message | Deliver the corrected claim only; the trace holds the error |
| `It's probably the retry storm.` | Hedge as conclusion | Act | `The retry storm saturates the pool at 128 waiters (14:10–15:00 UTC).` |
| `So, to summarize what I found: there are a few things going on.` | Recital | Rank the findings | One verdict line, then the findings ordered by blast radius |
| `I grepped for maxRetries and found…` | Instrument talk | Verify the finding independently | `\`retry.ts:41\` sets \`maxRetries: Infinity\`` |
| `I read the whole handler to be sure.` | Route as evidence | Trust or check the claim | `\`handler.ts:188\` fans out one call per line item` |
| `In this section, I'll walk through the changes.` | Meta-discourse | Skip anything | Delete. Open with the change ([Skim-Test Outliner](../../kirby-fitzpatrick-skim-test-outliner/SKILL.md)) |
| `I think it might be related, though I'm not certain.` | Narrative doubt | Rank or falsify the hypothesis | `Inferred: real but not causal — predates the 14:02 step change` |
| `Not 100 % sure this is safe to revert.` | Unlabelled belief | Decide whether to revert | `Unverified: safe unless a schema write occurred after 14:02 (check \`ledger_events\`)` |
| `Let me know if you want me to keep investigating.` | Deferred verdict | Know what to ask for | `Open: the N+1 at \`pricing.ts:62\` — separate ticket, not on the critical path` |

---

## 3. Engineering Application Scenarios

### Scenario A — Code Reviews (reviewer-side): the verdict is the comment

A review thread is a delivery channel with the same interference problem compressed into minutes, and the cost is a **round trip**. A comment that opens with process — *"I was looking at the retry logic and I'm trying to understand why…"* — forces the author to read the whole comment to learn whether they are blocked, and then to re-derive the reviewer's model from scratch. The review's Reader's Draft is `[VERDICT] + [CLAIM] + [LOCUS] + [PROOF]`, and the reviewer's own exploration (the reads, the false starts, the "let me check whether this is called elsewhere") stays in the reviewer's head or in the tool trace.

```text
ANTI-PATTERN — Comment as thinking aloud
────────────────────────────────────────────────────────────────────────────────
"Looking at the retry change here, I was wondering what happens if the
downstream stays down for a while. It looks like it might just keep going?
I'm not sure if maybe there's a cap somewhere else that I didn't see. Also
the naming of getLedgerV2 feels a bit off, minor though. And I checked the
migration ordering and I think it's fine, but you may want to double check."
```

```text
PATTERN — Comment as Reader's Draft  (skimmable, actionable, cited)
────────────────────────────────────────────────────────────────────────────────
1. 🔴 Blocking — retry has no ceiling; a 30-min downstream outage becomes
   ~40k QPS of retry traffic. `checkout/retry.ts:41` sets
   `maxRetries: Infinity` and no other cap exists (`rg -n "maxRetries" → 3
   hits, 0 in services`). Cap at 5 attempts + jitter.
2. 🟡 Non-blocking — `ledger_id` is string-concatenated (`ledger.ts:88`);
   collision risk above `tenant_id` 2^53. Use the composite key type.
3. ⚪ Nit — `getLedgerV2` → `fetchLedgerV2` for suite naming parity (L19).

Checked and clear: migration ordering (additive nullable column, no
backfill), so nothing to do there.
```

The `Checked and clear` line is the one place a reviewer's private exploration *earns* the channel: it is not narration, it is a **negative result** that prevents the author from re-doing the review. Negative results are Reader's Draft material when — and only when — the reader would otherwise have asked.

### Scenario B — PR Descriptions (author-side): the diff's Reader's Draft

A PR body is read thousands of times by people who cannot afford to read it once: on-call responders bisecting at 03:00, release managers scanning for blast radius, and you, in eight months, with none of your present state. The most common interference leak here is the **discovery narrative** — the PR that tells the story of how the bug was found — and the second most common is the **scratchpad as commit log** (`wip`, `actually fix it`, `oops`).

```text
ANTI-PATTERN — PR as journey log
────────────────────────────────────────────────────────────────────────────────
Title: "Fix ledger stuff"
## Summary
  At first I thought this was the scheduler, so I dug into that for a while,
  but it turned out not to be. Then I realized both services were writing the
  same table, so I added an idempotency key. There's also a bunch of cleanup
  and a refactor while I was in there. See commits for details.
## Test Plan
  Ran it locally, seems fine. CI is green.
```

```text
PATTERN — PR as decoupled delivery
────────────────────────────────────────────────────────────────────────────────
Title: Reject duplicate ledger postings with a composite idempotency key

## Cause — reconciler re-posted all in-flight batches after a mid-batch restart
  `post_batch()` (`ledger/worker.ts:141`) had no idempotency key, so a
  restart replayed the batch into fresh postings: 1,204 duplicates in the
  2026-01-14 incident window. Not the scheduler (ruled out below).

## Change — composite `(tenant_id, batch_seq)` key, written in-txn
  The key is inserted in the same transaction as the posting; a duplicate
  attempt returns the original posting. One new unique index, added online.

## Blast radius — write path only; read path and schema are unchanged
  No migration. Previous index build took 41 s in staging.

## Measured — replaying the incident window produces 0 duplicate postings
  12,004-batch replay of 2026-01-14; 0 duplicates vs. 1,204. Repro:
  `tests/replay_incident.py`.

## Rollback — revert the commit; the new index stays but is inert
  Key enforcement sits behind `IDEMPOTENT_POSTS` (default on).

## Rejected — scheduler duplication
  `scheduler/dispatch.ts:64` reads the batch once per tick; two ticks
  overlapped only during the 14 s restart, where the reconciler held the
  lock. Not the mechanism.
```

15-second reviewer path: title → `Blast radius` → `Rollback`. The reviewer's decision is made without touching the diff; the body exists for the implementer and the sceptic. The commit log is where the `wip`/`oops` sequence belongs — it is a legitimate trace, and it is *not* the deliverable.

### Scenario C — Architecture RFCs and ADRs: publish the decision, relocate the discovery log

An RFC is where the interference leak is most expensive, because the reader is a decider with a fixed budget, and because a discovery log looks like diligence — so it survives review. The RFC is the *decision*: constraints, decision, consequences, rejected alternatives, rollback. The exploration — the four candidates you benchmarked, the spike that failed in staging, the two weeks of reading — is not the decision, and pasted into the normative body it buries the decision under a recital. It belongs in a clearly marked appendix.

```text
ANTI-PATTERN — RFC as development diary
────────────────────────────────────────────────────────────────────────────────
# ADR-0042: Ledger Storage Strategy
## Status        Proposed
## Context
  We started by looking at Kafka, then DuckDB, then ClickHouse. I ran a
  spike on Kafka which was interesting and mostly worked, and DuckDB was
  surprisingly fast for the read path, though I'm still not sure about the
  write path. I also read a few posts about single-writer ledgers. After
  all of that, I think we should probably go with a single writer, but I'm
  open to other ideas.
## Consequences  There are trade-offs to consider.

SKIM RESULT (15 s): "There's a proposal. Which one? Unknown."
DECISION: blocked. One round trip, plus the reviewer re-litigates Kafka.
```

```text
PATTERN — RFC as Reader's Draft  (skim layer decides; appendix optional)
────────────────────────────────────────────────────────────────────────────────
# ADR-0042: Single-writer ledger on v3; v2 becomes read-only on day 8
Status: Proposed · Owner: @ledger-team · Open: cutover date (not the design)

## Decision — one writer, one table; ledger id is the idempotency key
  Removes the dual-writer race by construction, not by locking.

## Cause — v2's two writers race on stale reads; 1,204 duplicates on 2026-01-14
  Per-writer fixes failed 3× (#8811, #8840, #8902): the race is in the
  shared table, so it cannot be fixed at either writer.

## Estimated — dual-write adds ~38 % p99 write latency for 7 days (+/- 9 %)
  From the 2026-01-20 shadow run at 11.8k rps. Above 12k the buffer queue
  is expected to saturate — that is the risk boundary.

## Consequences — reports lag ≤ 1 day for 7 days; v2 schema frozen meanwhile
## Rollback — `LEDGER_PRIMARY=v2`, ≤ 60 s replay, no data loss
## Blocking — 340 contract fixtures must pass on v3 before cutover (@ledger-team)
## Rejected — per-writer locking on v2 (deadlocks at 12k rps in the shadow run)

---
### Appendix A — Discovery log (non-normative; not required reading)
  Spikes: Kafka (rejected: 4th stateful dep for a 900 msg/s peak), DuckDB
  (read path viable, write path unmeasured), ClickHouse (schema cost).
  Chronology, raw benchmarks, and the failed staging cutover: `spike/`
  branch, tag `adr-0042-spikes`.
```

The appendix is the point: the scratchpad is *preserved and routed away*. A reviewer who wants the diligence can have it; a decider who wants the verdict never sees it. Number the appendix as non-normative and keep it out of the decision path — an appendix that changes a decision is a body, and it has been mis-filed.

---

## 4. Verification Checklist

- [ ] **The verdict is in the landing zone, in the asker's words.** The first sentence of every delivered message answers the question asked, in decisive mood, ≤ 25 words, with no preamble about my process, my plan, the question itself, or "in this section". I verified by reading only the first three lines and asking whether a reader who stopped there could act.
- [ ] **Zero channel crossings.** No delivered sentence describes my process, tools, or intentions: no `Let me…`, `I'll check…`, `First I looked…`, `Actually, wait`, `Hmm`, no self-correction trail, no second summary of the same mechanism. The count of such sentences is 0, not "few" — and I can show the trace separately if asked.
- [ ] **Order is decision order, not discovery order.** Load-bearing claims lead (*what happened → why → proof → boundary → undo → open*); every abandoned branch was converted to a committed claim, a one-line `Rejected:` with its falsifier, or deleted. No delivered text narrates the journey that produced it.
- [ ] **Every claim carries evidence or a status prefix — never a narrative hedge.** Each load-bearing statement cites `path:line@sha`, a named measurement with its window, or an explicit `Measured:` / `Inferred:` / `Unverified:` / `Blocked:` prefix with its falsifier; the words *probably, seems, maybe, I think, clearly, obviously* appear zero times in load-bearing positions; and no claim in the delivered message rests on a probe that is not also cited there.
- [ ] **The scratchpad was relocated, and the delivery passed the deletion test.** The exploration (probes, dead ends, chronology) is preserved in a trace, log, commit body, or appendix marked non-normative — never destroyed, never in the reply. Every sentence that survived deletion changes what the reader will do or believe; `delivery yield ≥ 0.90` on final answers, `≥ 0.70` on status messages, and each turn of a multi-turn task delivered its own verdict-or-status plus next action rather than deferring to a closing summary.