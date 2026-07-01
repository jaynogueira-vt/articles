# Incident Post-mortem (Hermes skill)

A method for writing an incident post-mortem that is grounded in real evidence,
matches your team's template exactly, and reads as a durable engineering record
rather than a transcript of the war room. It codifies the one discipline that
separates a trustworthy post-mortem from a plausible-looking one: **every factual
claim is backed by a durable artifact, and anything that cannot be fact-checked is
cut, not softened.**

> Drop this into `skills/incident-postmortem/SKILL.md` in any agent that supports
> skills (or your skills dir) and adapt the template structure and publish steps
> to your own wiki (Confluence, Notion, a Markdown repo, wherever your team keeps
> post-mortems).

## What it prevents

- A post-mortem whose Root Cause is built on hypotheses people floated live
  while debugging, half of which turn out to be wrong.
- A "5 Whys" that is actually a paragraph describing the cause, so the real root
  cause is never reached.
- Blame framing, approximate numbers, and meta-narration ("this doc aims to...")
  that make reviewers distrust the whole document.
- A published record that goes stale the moment someone re-reads it, because it
  quoted the incident channel instead of the code, the schema, or the ticket.

## When to use (auto-load triggers)

Load this when the task is any of: "write / create the post-mortem", "the
existing post-mortem is bad, redo it from scratch", "document this incident", or
whenever an incident has been mitigated and needs a written record within the
usual one-business-day window.

## Evidence discipline (the non-negotiable core)

**A meeting transcript or incident-channel scrollback is a GUIDE, not a SOURCE OF
TRUTH.** Triage conversations are full of live hypotheses people voiced while
debugging. They are leads to verify, not facts to publish. Every factual claim in
the post-mortem, and especially in Root Cause, must be backed by a durable
artifact:

- code or schema you can open and read,
- a merged pull request,
- a feature-flag / config console,
- a tracker ticket,
- a message someone actually wrote (not a paraphrase of "the room thought..."),
- production data, or an attached log / query / export.

**If a claim cannot be checked against such an artifact, remove it.** Do not
soften it, do not attribute it to "the war room." Just cut it. A short,
fully-evidenced Root Cause beats a complete-looking one built on hearsay.

Process when drafting Root Cause from a transcript:

1. List every causal claim the transcript suggests.
2. For each, find the durable artifact that proves it: grep the schema, open the
   PR, check the flag, find the message. A missing-constraint or wrong-default
   class of claim is usually checkable directly in the schema or config.
3. Keep only what you proved; cut the rest.
4. In the Evidence section, never list an artifact you have not confirmed exists
   (no "the failure logs" if nobody ever produced logs). If a plausible claim had
   no backing artifact, say explicitly that it was left out pending evidence.

## What a post-mortem is (and is not)

A post-mortem is a **lightweight COE** (Correction of Error), not a full COE:

- Do it immediately after mitigation, or the next business day if out of hours.
- Spend the majority of the post-incident meeting on **Root Cause**.
- Publish within about one business day of mitigation.
- A "Full COE Required?" decision at the end determines whether a heavier
  investigation follows.

## The five-section structure (plus a recommended sixth)

Do not invent or reorder sections. Match your team's template; the canonical
shape is:

1. **Intro panel**: one line identifying the incident and linking the incident
   ticket (as a described link, never a bare ID).
2. **Metadata**: a compact two-column table: Team, Started, Detected, Resolved,
   Detected By, Incident Owner, Engineering Owner, Other Attendees. Attribute the
   team that owns the *changed code* distinctly from the team that was *impacted*.
3. **Summary**: a short multi-paragraph narrative: what went wrong, what the team
   had to do, what was done, and the reassuring close (e.g. "delayed, not lost; no
   incorrect side effects"). Not a bullet dump.
4. **Impact**: blast radius in exact terms: how many users / records / sessions,
   over what window, and time to recover. Labeled analytical lines, not a flat
   list.
5. **Root Cause**: the most important section. See below.
6. **Action Items**: a short preamble, then **Short-term** and **Long-term**
   subsections, each a numbered table with **one ticket per row**, every ticket a
   described link. Apply your tracker's remediation labels (commonly `coe` for
   short-term and `coe_longterm` for long-term) so the items are trackable.
7. **Full COE Required?**: yes/no plus an explanation that defends the decision.
   If the root cause is fully understood and the action items are concrete, you
   usually do not need a full COE.
8. **Evidence** (recommended): the durable artifacts behind every claim, grouped
   under bold labels (Incident tracking, Response and timeline, Triggering change,
   Code touchpoints, Remediation). This is what makes the evidence discipline
   auditable: every claim in Root Cause and Impact should trace to a link here.
   List only artifacts you confirmed exist.

## Root Cause is a 5 Whys exercise, and nothing else

Root Cause **must** contain an actual **5 Whys**, not prose that describes the
cause. Write it as flowing `Why [question]? Because [answer].` pairs that chain
the symptom down to the true root cause, with the final step being the root
cause. A paragraph that merely narrates the cause does not satisfy this and a good
reviewer will bounce it.

**Root Cause has no subsections.** It is the 5 Whys and nothing else. Do not add
"Contributing factors", "Mitigation", or "Ruled out" headings. Mitigation belongs
in the Summary narrative, and a genuinely important dismissed hypothesis goes in
one sentence at the end of the relevant Why, not a standalone section.

Separate the **trigger** (the change that set it off) from the **amplifier** (what
made it worse). Keep upstream responsibility clear but never accusatory: a 5 Why
ends at the technical mechanism, never at organizational fault. State a recurrence
as a neutral fact ("the fourth occurrence from the same migration"), not a
grievance.

## From-scratch build workflow

The common trigger is "the existing post-mortem is bad, redo it from scratch,"
which means a full evidence build, not a reword. Gather in parallel:

1. **Read the existing doc and all its comments.** The comments are reviewers
   telling you what is missing. That is your must-fix list.
2. **Resolve every input link.** Identify upstream vs downstream incident docs and
   cross-link them.
3. **Find the incident call transcript(s)** and extract the timeline, decisions,
   and who did what, as leads to verify, per the evidence discipline above.
4. **Pull the tracker**: incident tickets, short-term fix tickets, long-term epic.
5. **Read the repo(s)** for what actually shipped as the short-term fix (PRs) and
   what is planned long-term (epic + design doc).
6. **Get exact numbers from production**: logs or the database, not estimates.
7. **Draft into the template**, grounded in evidence, and review with a human
   before publishing.

## Hygiene rules (a reviewer will bounce the doc otherwise)

- **Exact, verified numbers.** Every measurement (affected users / records,
  ticket counts, recovery time, the measured shortfall) must be exact and pulled
  from logs or the database. No "and climbing." An approximate `~` is acceptable
  **only** on an inherently-estimated external expectation or benchmark, and only
  when you name the source of the estimate in the same sentence. If you can
  measure it, measure it.
- **Links carry a short description, never a bare ID.** Every ticket, PR, and
  prior post-mortem is a hyperlink whose visible text is a few descriptive words,
  never the raw key or number. `Add the missing foreign-key constraint` linked to
  the ticket, not `PROJ-1234` as the label, and not `PROJ-1234:` as a prefix.
- **One timezone, stated.** Pick your team's canonical timezone and use it
  consistently; never mix in UTC silently.
- **No blame.** Attribute who owns the changed code vs who was impacted, factually
  and minimally, in the metadata, not as a grievance in Root Cause.
- **No meta / process / self-narration.** The doc contains incident facts only,
  never commentary about how the doc was written ("data first, since that's what
  readers want", "as requested", "the prior version"). If data goes first, just
  put it first; do not announce that you are doing so.
- **No duplication.** The reassuring "no data lost / no incorrect side effects"
  line tends to repeat, so say it once.
- **Keep it durable.** No local file paths, temp paths, raw logs, tokens, or
  secrets in the body. Everything should still make sense a year later.

## Publish, then verify

- Build the document body, save an exact copy locally for recovery, run the
  hygiene scan (numbers, links, timezone, blame/meta language), then publish.
- **Version-drift trap:** if a human may be editing the live page while you work,
  re-fetch it immediately before you write and check the version. If it advanced
  past the version your edit was based on, re-derive your change on the fresh copy
  rather than blindly overwriting, or you clobber their edits (column
  widths, label fixes, and so on).
- After publishing, re-fetch the stored form and re-run the hygiene checks on it
  (wiki engines silently transform characters on round-trip), confirm every
  section and every action-item link resolves, and probe each user-facing URL.

## Pitfalls

1. **Transcript claims leak into Root Cause.** The single most common failure.
   "Event logs confirm a failure", "a migration script was never applied", "an
   error in the script itself": all classic transcript hypotheses that sound
   authoritative and turn out to be unverifiable or wrong. Verify each against an
   artifact or cut it.
2. **The 5 Whys is really a paragraph.** If it does not chain question →
   because → question, it is not a 5 Whys.
3. **Approximate numbers on measurements.** `~3,600 affected` reads as "we didn't
   check." Measurements are exact; only sourced external expectations may be
   approximate.
4. **Bare IDs as link text.** A clickable `PROJ-1234` still reads as a bare ID.
   The label must be a human description.
5. **Blame or meta language.** Both quietly destroy trust in an otherwise correct
   document. Scan for them before publishing.
6. **Short-term lists forced to match across paired docs.** When an upstream and a
   downstream post-mortem share a long-term fix, keep that long-term item identical
   and concise across both, but let the short-term lists differ, because they are
   different incidents with different immediate fixes.

## Tools this skill uses

- wiki / docs API: read the existing doc and its comments, publish, re-fetch to
  verify
- tracker API: pull incident, fix, and epic tickets; apply remediation labels
- file/content search + file read: open the code and schema that prove or
  disprove each causal claim
- terminal: inspect merged PRs, run the hygiene scans, query production for exact
  numbers
