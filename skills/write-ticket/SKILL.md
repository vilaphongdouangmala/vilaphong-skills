---
name: write-ticket
description: Turn raw material (requirements, a client's bug report, meeting notes) into a paste-ready GitLab ticket that passes the Codium Ticket Convention's Definition of Ready.
disable-model-invocation: true
---

# Write Ticket

Turn whatever the user hands you — a requirements dump, a client's reported bug, a chat excerpt, a half-sentence — into one paste-ready ticket that passes the **Definition of Ready**.

The full convention ships with this skill at [`TICKET_CONVENTION.md`](TICKET_CONVENTION.md). Read it when a case here doesn't cover you — board hygiene, the rejection flow, `Needs::` semantics, and the reasoning behind every rule live there.

**The one rule:** a ticket says **what should be true when it's done**, never how to build it. No file paths, no function names, no "add a service method". That's the developer's job.

**Never invent.** Every fact in the output must trace to something the user said or confirmed. A gap is a question, not a guess — see step 2.

## Process

### 1. Classify

Decide `Type::` from the raw material. This drives which body template you use, so it comes first.

| Type | When |
|---|---|
| `Type::Bug` | Something already delivered behaves wrong. Uses the **bug body**. |
| `Type::Feature` | New implementation on a from-scratch project — no shipped baseline to change yet. **Standard body**. |
| `Type::CR` | New or changed user-visible behavior on an existing product, billed to the client. **Standard body**. |
| `Type::Free Request` | Same work as a CR, not billed. **Standard body**. |
| `Type::Chore` | Necessary work, no user-visible change (upgrades, cleanup, CI). **Standard body**. |

Three rules: **a bug is never a Free Request** — if we delivered it broken it's `Type::Bug` no matter who absorbs the cost. **CR vs Free Request is a billing decision only the user can make** — you cannot infer it from the work, so it is always a question in step 2 unless the user already said which. And **`Type::Feature` isn't an escape hatch from that billing call** — reach for it only when there's no existing delivered product being changed; the moment there's a shipped baseline, it's a CR or Free Request.

Bug vs CR vs Feature is often genuinely ambiguous ("the report doesn't show tax" — was tax ever specified, and does this product even exist in production yet?). When you can't tell from the material, ask; don't default.

**Done when:** you hold a single `Type::`, either read off the material or answered by the user.

### 2. Ask every gap, in one round

Read the material and list what the ticket is missing. Then ask it all in **one** `AskUserQuestion` round (up to 4 questions; if you have more than 4 gaps, ask the 4 that most change the output, then a second round). Give options where the answer space is closed (Type, Priority, billing), free text where it isn't.

Hunt these gaps specifically:

- **Type** — if step 1 left it open, or the CR/Free Request billing call is unmade.
- **Priority** — `P0` production down · `P1` next release or hard deadline · `P2` default · `P3` nice to have. If the user gave no urgency signal, propose `P2` as the recommended option rather than asking blind. `P0` is for outages, not for a client asking loudly.
- **Area** — the title's `<Area>:` prefix. Whatever the project already calls that screen or module. If the material doesn't name one, ask.
- **The outcome** — what should be true when it's done, in the user's terms. If the material only describes a problem and not a resolution, that's a gap.
- **Why now** — who asked, what it unblocks. Cheap for the user to answer today, expensive in six months.
- **Bug-specific:** reproduction steps, expected behavior, environment (version, device/browser, prod vs UAT vs dev), evidence. A bug with no repro steps is `Needs::Info`, not a ticket — so ask, and if the user genuinely doesn't have them, say so in the output rather than fabricating a sequence.
- **Failure cases** — anything touching input validation, money, permissions, or an external API has them. Ask what should happen when it goes wrong; don't decide on the user's behalf.
- **Estimation** — if the user needs a man-day estimate before this can be committed to a release and doesn't have one, that's `Needs::Estimation`, not a blocker on writing the ticket itself.

Skip a question when the material already answers it. Asking about a fact the user just gave you reads as not having read it.

**Done when:** every unknown you can see is either answered or explicitly flagged in the output as an open question. Nothing is filled in by assumption.

### 3. Draft the body

Use the template matching the `Type::` from step 1. Delete optional sections you have nothing for — an empty heading is worse than no heading.

<standard-body>

```markdown
[Figma](<link>)

## Background
<!-- 1-2 lines: why now / who asked, then what should be true when it's done. -->

## Flow
<!-- Optional. Write it for anything with steps. One block per flow. -->

## Acceptance Criteria
- [ ]
- [ ]

## Out of Scope

## Notes / Constraints
```

**Background** is required and is **1-2 lines**: why now, then the outcome. Lead with the outcome if you only get one line. If the outcome needs more than a sentence, tell the user it's probably two tickets.

**Flow** — optional, but write it whenever the material describes steps; a numbered flow shows *sequence*, which a checklist can't. One block per trigger path (an API path and a UI path are two flows, not one flow with branches). Steps are user actions and system responses. End every flow with the observable result — "Status changes from Approved to Cancelled", not "user clicks Confirm". Flows are the happy path; **failures go in AC**.

**No "Items to implement" list.** If you catch yourself writing "add button X, make field Y required", convert it to an outcome in AC ("cancelling without a reason is rejected"). It duplicates the Flow and goes stale.

**Out of Scope** — use it when the ticket sits next to something it could sprawl into.

**Notes / Constraints** — facts, not instructions. "The client insists the button is on the right" is a constraint. "Add a flag to the Order model" is not.

</standard-body>

<bug-body>

```markdown
## What happens
<!-- Observed behavior. -->

## What should happen
<!-- Expected behavior. This IS the acceptance criteria. -->

## Steps to reproduce
1.
2.
3.

## Environment
<!-- App version, device / browser, environment, user account or role. -->

## Evidence
<!-- Screenshot, recording, log excerpt, request ID. -->
```

No separate AC section on a bug — "What should happen" is the AC. Client bug reports arrive as one blurred sentence; your job is splitting it into observed vs expected, which is usually where the real ambiguity surfaces.

</bug-body>

**Title**, both types: `<Area>: <what the user can do / what changes>`, under ~70 characters. `Checkout: apply promo code at payment step`, not `Fix checkout` and not `Update UserService.applyDiscount()`.

**Acceptance Criteria** — one test, and only one:

> Could an outsourced QA tester who has never seen the code read this line and decide pass or fail, without asking anyone?

Plain sentences pass. There is no required format. Reach for `Given <situation>, when <action>, then <result>` only where the *starting state* matters and is easy to forget ("given a code is already applied…"), and mix formats freely in one ticket.

One line is a complete AC for a small ticket — don't pad. Past ~10 lines it's two tickets. Every line must be checkable: "works correctly" and "handle errors" fail the test; "an expired code shows an inline error and the total is unchanged" passes.

**Done when:** every section is either filled from confirmed facts or deleted, and every AC line passes the QA-tester test.

### 4. Check it against Ready, then output

Verify each item, and say which ones the ticket does **not** yet meet:

- [ ] Title follows `<Area>: <outcome>`
- [ ] Background filled in — why now, and the outcome, in 1-2 lines
- [ ] Acceptance Criteria written, each line independently checkable
- [ ] `Type::` and `Priority::` set
- [ ] Designs / API contracts linked if the work needs them

Then output exactly three blocks, nothing else:

1. **Title** — one line, plain text, ready to copy into the title field.
2. **Description** — one fenced markdown block, ready to paste into the description field.
3. **Labels** — one line: the `Type::`, `Status::`, and `Priority::` to apply. `Status::To Do` when the ticket passes Ready; `Status::Backlog` when it doesn't. Add `Needs::Info` when something is still missing (typically a bug with no repro steps), or `Needs::Estimation` when the ticket is otherwise Ready but is waiting on a man-day estimate before it can be committed.

> **Gotcha — never trust `glab label list` for the full label set.** It paginates at ~30 and silently truncates, hiding whole families like `Type::*` or `Status::To Do` — a "looks authoritative, actually incomplete" trap that leads to shipping wrong labels or inventing ones that already exist. Always enumerate labels with the paged API instead:
> ```bash
> glab api "projects/<url-encoded-path>/labels?per_page=100"
> ```
> Confirm the exact `Type::`, `Status::`, and `Priority::` labels exist there before applying them.

If the ticket lands in `Backlog`, state in one line what it's waiting on. That line is the difference between a ticket someone can finish and one that quietly stalls.

**Done when:** the user can paste the title and description into GitLab without editing either, and knows which labels to click.
