# RFC Authoring (Hermes skill)

A method for writing an engineering RFC (design doc) that is an implementation
contract, not a product brief wearing an RFC hat. It encodes a canonical section
structure synthesized from public sources (Google Design Docs, Phil Calcado's
structured RFC process, Oxide's RFD tradition), a firm boundary between product
scope (WHAT/WHY) and design scope (HOW), per-section depth anchors that separate
real design content from placeholder prose, and a research-first rule: verify
your infrastructure claims against the codebase before you assert them.

> Drop this into `skills/rfc-authoring/SKILL.md` in any agent that supports
> skills (or your skills dir) and adapt the publish steps to wherever your team
> keeps design docs (a wiki, a docs repo, a shared drive).

## What it prevents

- An "RFC" whose sections have the right names but product-brief content, so
  engineering review turns into product debate.
- Confident infrastructure assertions that turn out wrong (naming the wrong
  framework, assuming a queue or an event that does not exist), discovered by
  reviewers instead of by you.
- An empty Alternatives section ("we picked the obvious thing"), which makes the
  design indefensible under review.
- Rollout plans without a kill switch and state machines without invariants.
- Open Questions used as a parking lot for things you have not read yet.

## When to use (auto-load triggers)

Load this when the task is any of: "write an RFC", "design doc", "technical
spec", "implementation design", or when a product requirements doc is approved
and the next step is engineering design. Also load it when asked to research RFC
formats: the synthesis is already here.

## When to write an RFC (and when not to)

Write one when:

- A non-trivial change touches more than one service, surface, or team, and
  reviewers need to weigh trade-offs before code exists.
- An approved product doc needs an implementation contract before work starts.
- The design has at least one consequential decision where the rejected
  alternatives matter (data model, state-machine semantics, sync vs async).
- The change introduces or modifies persistent state (schemas, financial
  records, audit trails), or rollback and migration are non-trivial.

Do not write one for: single-file refactors, bug fixes (a ticket plus a PR
description is enough), decisions scoped to one team with no external surface
(consider an ADR), or anything the product doc already commits to verbatim.

## The firm boundary: product doc vs RFC

The most common authoring failure is bleeding product content into the RFC or
design content into the product doc. Hold the line:

| Concern | Product doc (WHAT/WHY) | RFC (HOW) |
|---|---|---|
| Audience | PMs, leads, legal, finance | Engineers, tech leads |
| Language | Domain ("customer accepts the offer") | Technical (table names, endpoints, state machines) |
| Event names | "We need to measure acceptance rate" | `offer_accepted` event with properties |
| Vendor / tooling | Out of place in the body | Required in Observability |
| `snake_case` identifiers | Out of place | Standard |
| Timeline | Launch window only | Full staged rollout plan |
| Open Questions | Policy and scope decisions | Design and implementation decisions |

If you catch yourself writing event schemas in the product doc, move them to the
RFC. If you catch yourself writing rollout-percentage targets in the RFC, move
them to the product doc.

## Canonical section structure

Eleven top-level sections, in this order. Do not invent, rename, or reorder
top-level sections; extend inside them instead.

```markdown
# RFC: <Feature Title>

**Parent epic / tracker link:** ...
**Companion product doc:** ...
**Author / Status / Reviewers / Last updated:** ...

## 1. Summary
One paragraph, five sentences max: what this proposes, the high-level
approach, and the single most consequential design decision.

## 2. Context & Background
Why this exists. The existing surfaces this will modify, in technical
terms. Prior RFCs or ADRs that constrain the design.

## 3. Goals & Non-Goals
Technical goals, each traceable to a requirement or a named concern.
Non-Goals: things a reasonable reviewer would expect but that are
intentionally out of scope.

## 4. Proposed Design
### 4.1 System Context (diagram naming real services and boundaries)
### 4.2 Data Model (schemas, constraints, indexes, migration plan)
### 4.3 API & Contracts (endpoints, shapes, auth, idempotency)
### 4.4 State Machine & Workflow (transitions, invariants, concurrency)
### 4.5 Error Handling & Recovery (failure modes mapped to recoveries)
### 4.6 Backwards Compatibility & Migration (flags, cutover, rollback)
### 4.7 Performance & Capacity (volume, latency budgets, headroom)

## 5. Alternatives Considered
Each alternative gets a name, a paragraph or two, and an explicit
rejection reason. "Do nothing" is always one of them.

## 6. Cross-Cutting Concerns
Security & authorization; privacy & compliance; observability (map each
plain-language success signal to a concrete event, metric, or span);
testing strategy.

## 7. Rollout Plan
Numbered stages with audience, duration, advance criteria, kill switch,
and a communication plan.

## 8. Open Questions
Only genuinely unresolved HOW-level decisions. Each: decision needed,
options with trade-offs, owner, deadline.

## 9. Approvals
Sign-off table (role, approver, date).

## 10. Revision History

## 11. Appendix
Related docs, glossary, references, full SQL if the body summarized it.
```

Allowed extensions: sub-headings inside section 2 when context spans groupings,
named sub-headings per alternative in section 5, rollout stages as a table, and
structured sub-fields per open question. Not allowed: new top-level sections
(put "Future Work" in the appendix or a follow-up RFC), renamed sections, or
promoting a 4.x sub-section to top level.

## Depth anchors: real design content vs placeholder prose

Headings alone do not make an RFC. For each section, if you cannot reach the
anchor, mark the gap explicitly (for example `[PENDING CODEBASE RESEARCH:
<specific question>]`) instead of papering over it with prose. A reviewer can
walk those markers as a worklist; that is honest scaffolding.

| Section | Shallow (reject) | Real depth (target) |
|---|---|---|
| System context | "there is a frontend and a backend" | a diagram naming actual services, the boundary the new code crosses, and what the feature flag gates |
| Data model | "we'll need a table" | `CREATE TABLE` with types, constraints, indexes, uniqueness that enforces idempotency, and the migration tooling named |
| API | "user can accept or reject" | method + path + headers + status codes for the happy path and each failure mode, request/response shapes, the auth rule |
| State machine | a bullet list of states | transitions plus at least three named invariants, the concurrency strategy, and how timeouts fire |
| Error handling | "we handle errors gracefully" | a table mapping each edge case to a concrete recovery, plus failure modes specific to this design |
| Rollout | "behind a flag, gradual" | flag name and scope, numbered stages, what happens to in-flight entities at each transition, explicit kill-switch behavior |
| Alternatives | one option plus "we picked the obvious one" | three or more, each named, described, and explicitly rejected; "do nothing" included |

The smell test before publishing: for every design paragraph ask "could a senior
engineer start coding from this, or would they need to ask five more questions
first?" If it is five questions, the section is still product-shaped.

## Research first, prose second

Pending-markers do not protect you when the surrounding sentence is itself a
guess. Before drafting declarative prose, run a research pass over the actual
repos and systems the design touches: the backend and frontend code, the
producer side of every event you consume, the domain glossary, the observability
stack that actually exists. Classic failures this prevents: calling an app the
wrong framework, assuming an outbox pattern that does not exist, consuming an
event no producer emits, and inventing state names that collide with an existing
domain glossary. A claim you could not verify gets tagged
`[INFERRED: verify against <repo>/<file>]`, never asserted bare.

For every cross-service interaction, open the producer's repo and confirm the
event or endpoint exists before designing a flow against it.

## Self-review checklist (before sharing)

- [ ] Summary fits in five sentences and names the most consequential decision.
- [ ] Goals trace to requirements; Non-Goals exists and is non-empty.
- [ ] The design section leads with a compact diagram, not a wall of code.
- [ ] Full SQL lives in the appendix; the body summarizes schemas as tables.
- [ ] Alternatives: at least "do nothing" plus two others, each with an explicit
      rejection reason.
- [ ] Observability maps every success signal to a concrete event or metric.
- [ ] Rollout names the flag, the monitoring gates, and the rollback path.
- [ ] Open Questions were checked against the source docs and comments first;
      anything already answered moved to a resolved-decisions table.
- [ ] Every ticket and doc reference is a described hyperlink, never a bare ID.
- [ ] Under roughly 6,000 words. Longer means it is either a spec masquerading
      as an RFC or two RFCs in one.
- [ ] Every infrastructure assertion traces to research or carries an
      `[INFERRED]` tag.

## Resolving open questions (do not just park them)

When an open question is empirically answerable from the codebase, resolve it
yourself: gather evidence (file and line citations, grep counts, producer-side
confirmation), write the resolution with a revisit trigger, and have a second
model or reviewer cross-check the verdict. Once every entry is resolved, rename
the section "Design Decisions": a section titled Open Questions full of decided
items misleads reviewers. Keep the numbering so cross-references survive.

Two ownership rules while resolving:

- Do not silently file tickets as a substitute for investigating. A ticket
  defers what an hour of code reading can settle.
- Do not edit the companion product doc, even if you authored it. If you find a
  product-level fact that is wrong, flag it to its owner; the fix lands there,
  not in your RFC.

## Pitfalls

1. **Product content bleeding in.** The single most common failure. Re-read the
   boundary table; move vendor names and identifiers into the RFC, move policy
   and metric targets out.
2. **Empty Alternatives.** Indefensible under review. Name "do nothing" and at
   least two real options.
3. **Observability as afterthought.** If success signals never map to concrete
   events, the product doc's metrics promise is uncashable.
4. **State machine without invariants.** A transition list is not a state
   machine. Name what cannot happen, what guarantees idempotency, and what
   concurrent conflicting transitions do.
5. **Rollout without a kill switch.** "We'll roll back via deploy" is not a
   kill switch.
6. **Overwriting collaborative edits when republishing.** If the doc lives in a
   wiki and you regenerate it from a local source, fetch and merge the live
   edits (status fields, reviewer mentions) before overwriting. The wiki is the
   source of truth for collaborative metadata; your local file is the source of
   truth for the body.
7. **Cross-reference drift.** When either doc's section numbers change, grep the
   companion for stale references and fix them, in both directions.
8. **An evidence dump with RFC headings.** Research-correct is not
   review-ready. Keep the body at the decision level (decision, rationale,
   implication); move raw evidence to the appendix. If a screenshot of the
   design section looks like a terminal dump, it is not ready.
9. **Asking open questions the source docs already answered.** Re-read the
   product doc, its comments, and the domain glossary before publishing the
   Open Questions section.
10. **Glossary drift.** Before naming new states, fields, or events, read the
    domain glossary. If a canonical term exists, use it; if you must invent,
    namespace clearly so it cannot collide.

## References (public sources this synthesizes)

- Phil Calcado, "A Structured RFC Process": https://philcalcado.com/2018/11/19/a_structured_rfc_process.html
- Design Docs at Google (Industrial Empathy): https://www.industrialempathy.com/posts/design-docs-at-google/
- Oxide, "RFD 1: Requests for Discussion": https://oxide.computer/blog/rfd-1-requests-for-discussion
- IETF RFC Editor (the original "Request for Comments" tradition): https://www.rfc-editor.org/

## Tools this skill uses

- file/content search + file read: verify every infrastructure claim against the
  actual code before asserting it
- terminal: grep the producer repos, inspect schemas and migrations
- wiki / docs API: publish, re-fetch, and merge collaborative edits safely
- a second model or reviewer: cross-check resolved open questions before they
  graduate to design decisions
