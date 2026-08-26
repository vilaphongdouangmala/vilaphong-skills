# Ticket Convention

How we write tickets for developers. Applies to every project, every team.

**Scope:** this document covers ticket *content* and *labels* only. Sprints, estimation,
and ceremonies are out of scope.

**Who this is for:** the Project Manager writes the ticket. The developer reads it and
should be able to start without a meeting.

---

## 1. Why we're doing this

We have one convention for the whole organization rather than one per project. That's the
point — the value is in it being **the same everywhere**, not in any single rule below
being clever.

### Goal 1 — A ticket should be understandable without a conversation

A developer should be able to open a ticket and know what to build and when it's done,
without messaging the PM first. A PM should be able to write one without a developer
explaining the format. Every "quick question" about a ticket is time neither person
planned to spend, and it happens more often on unclear tickets than anyone tracks.

### Goal 2 — A PM moving between projects shouldn't have to relearn anything

Today, each project has its own habits: different labels, different statuses, different
idea of what a ticket needs. A PM switching projects has to learn all of it again, and
until they do, their tickets look wrong to that team.

With one convention, moving projects means learning the *domain* — the product, the
client, the codebase. That's the part that genuinely can't be standardized. The mechanics
of ticket writing shouldn't cost anything on top.

The same applies to developers working across projects, and to outsourced QA, who
otherwise need a per-project briefing before they can test anything.

### Goal 3 — A new PM should be able to see what to do next

A new PM should get up to speed from the board and this document, not from someone's
memory. Concretely, they should be able to answer without asking anyone:

- What am I supposed to fill in? → the template (§4)
- Is this ticket ready to hand to a developer? → Definition of Ready (§8)
- What's blocking this ticket right now? → its `Status::` and `Needs::` labels (§7)
- What should the team work on next? → `Priority::` (§7.3)

This also makes handover survivable. When a PM leaves a project, the state of the work
is on the board rather than in their head.

### What this is not

This isn't a bid for more process. Every rule here has to earn its place, and §10 includes
cutting whatever doesn't. If a section of this document is being filled in as "N/A"
project-wide, that's evidence against the section, not against the team.

---

## 2. The one rule

> A ticket describes **what should be true when it's done**, not **how to build it**.

The PM owns the *what*. The developer owns the *how*. If a ticket tells a developer
which function to edit, it's probably over-specified. If a ticket doesn't say how to
tell whether it worked, it's under-specified.

---

## 3. Title format

```
<Area>: <what the user can do / what changes>
```

Examples:

| Good | Why |
|---|---|
| `Checkout: apply promo code at payment step` | Area + user-visible outcome |
| `Login: block sign-in after 5 failed attempts` | Clear, testable |
| `Invoice PDF: show tax breakdown per line item` | Specific |

| Avoid | Why |
|---|---|
| `Fix checkout` | Fix what? Done when? |
| `Update UserService.applyDiscount()` | Tells the dev how, not what |
| `Promo code` | Not a change, just a noun |

Keep it under ~70 characters. `<Area>` is whatever your project already calls its
modules or screens; be consistent within a project.

---

## 4. Ticket body template

Copy this into every new ticket. GitLab: save it as a description template so it
auto-fills (see §10).

**Two sections are required, and one of them is 1-2 lines.** Delete the optional
sections if you have nothing to put in them — an empty heading is worse than no heading.

```markdown
<!-- Design / Figma link first, if there is one. -->
[Figma](<link>)

## Background
<!-- 1-2 lines. Why now / who asked, and what should be true when it's done. -->

## Flow
<!-- Optional but encouraged for anything with steps. One block per flow.
     End each flow with the observable result. See §6. -->

## Acceptance Criteria
<!-- How we'll know it works, including what happens when it goes wrong.
     Plain sentences are fine. -->
- [ ]
- [ ]

<!-- ---------- optional below: delete what you don't need ---------- -->

## Out of Scope
<!-- Anything you want to explicitly rule out. -->

## Notes / Constraints
<!-- API contracts, client-mandated behavior, deadlines, links. -->
```

### Section rules

**Background** — required, **1-2 lines**. Two things, in this order:

1. **Why now** — who asked, what problem it solves. This is what saves the ticket six
   months from now when nobody remembers why we built it this way. It's cheap to write,
   because you've just had the conversation.
2. **What should be true when it's done** — the outcome, in the user's terms.

Lead with the outcome if you only manage one line. That's the part a developer needs to
start; the history matters more later than it does today.

> **Background**
> Marketing's campaign starts 1 Sep and support is applying discounts by hand.
> A customer should be able to enter a promo code at payment and see the discount
> before paying.

If the outcome needs more than a sentence, that's usually a sign it's two tickets.

**Flow** — optional, but write it for anything with steps. Many of our tickets are
naturally described as user flows, and a numbered flow is often clearer than a checklist
because it shows *sequence*. See §6 for how to write them, including the case where one
feature has several flows (an API path and a UI path, say).

**Acceptance Criteria** — required. This is the one section with real thinking in it, and
it's the gate for `Status::To Do`. See §5 — the format is deliberately loose.

If you wrote a Flow, the AC section is where the **failure cases** go: the flow covers the
happy path, and AC covers what happens when it doesn't go to plan.

**No "Items to implement" list.** If you find yourself listing *"add button X, add popup Y,
make both fields required"*, stop — that's the developer's job, and it duplicates what the
Flow already said. Put the requirement in AC as an outcome instead
(*"cancelling without a reason is rejected"*, not *"reason field is required"*). Duplicated
facts go stale: when one copy changes, the other silently lies.

**Out of Scope** — optional. Use it when a ticket sits next to something it could easily
sprawl into. A developer may add to this during a scoping conversation; it doesn't have to
be written upfront by the PM.

**Notes / Constraints** — optional. Facts, not instructions. "The client insists the button
is on the right" is a constraint. "Add a flag to the `Order` model" is not.

---

## 5. Acceptance Criteria: the part that's new

We have not been writing AC. This is the biggest change in this document, so here is what
"good" looks like — and, just as importantly, how little is actually required.

**There is exactly one test for an AC line:**

> Could an outsourced QA tester who has never seen the code read this line and decide
> pass or fail, without asking anyone?

If yes, the line is fine. **There is no required format.** A plain sentence is enough:

```markdown
- [ ] Expired code shows an inline error and the total doesn't change.
- [ ] Applied code appears on the confirmation screen and in the receipt email.
```

Both of those are checkable, so both are good AC. Don't dress them up.

### When `Given / When / Then` helps

`Given <situation>, when <action>, then <result>` is a **tool, not a requirement**. Reach
for it when the *starting state* matters and is easy to forget:

```markdown
- [ ] Given a code is already applied, when the customer applies a second one,
      then it's rejected with "only one code per order".
```

Written as a plain sentence, "a second code is rejected" leaves the tester guessing what
the first code was doing. That's when the extra structure earns its keep. Otherwise skip it.

### How much is enough?

**One line is a complete AC** if the ticket is small. Don't pad to hit a number.

As a rough guide: if you can only think of one line for a ticket that clearly has failure
cases (anything with input validation, money, permissions, or an external API), that's a
signal to think about what happens when it goes wrong — not a rule that you owe five
bullets. More than about ten lines usually means it's two tickets.

### A small ticket — this is a complete, valid ticket

> **Title:** `Login: show password strength meter on sign-up`

```markdown
## Background
Support keeps seeing accounts with 4-character passwords. Users should see how strong
their password is while typing it on sign-up.

## Acceptance Criteria
- [ ] Strength indicator updates as the user types.
- [ ] Passwords under 8 characters are rejected with a visible reason.
```

No Out of Scope, no Notes, no Given/When/Then, four lines of writing. **This is not a
lazy ticket** — a developer can start on it and QA can verify it. Most tickets look like
this.

### A larger ticket — mixed formats

Note this one uses `Given/When/Then` only for the two lines where the starting state
matters, and plain sentences elsewhere.

> **Title:** `Checkout: apply promo code at payment step`

```markdown
## Background
Marketing campaign starts 1 Sep; support is applying discounts by hand today. A customer
should be able to enter a promo code at the payment step and see the discount before paying.

## Acceptance Criteria
- [ ] A valid code updates the order total and shows the discount as its own line.
- [ ] An invalid or expired code shows an inline error and leaves the total unchanged.
- [ ] Given a code is already applied, when the customer applies a second one, then it's
      rejected with "only one code per order".
- [ ] Given a code is applied, when the customer removes it, then the total returns to the
      original amount.
- [ ] The applied code appears on the confirmation screen and in the receipt email.

## Out of Scope
- Creating or managing promo codes (admin side) — separate ticket.
- Stacking multiple codes.

## Notes / Constraints
- Validation endpoint already exists: `POST /api/promo/validate` (see #412).
- Figma: <link>
```

Note what's absent from both: no table names, no file paths, no "add a service method".
The developer decides all of that.

### Common AC mistakes

The problem in every "weak" line below is that two people could disagree about whether
it passed:

| Weak | Stronger |
|---|---|
| Works correctly | An expired code shows an inline error and the total is unchanged |
| Handle errors | If the validation endpoint fails, the customer sees "try again later" and can still pay full price |
| Should be fast | The total updates within 1 second on 4G |
| Looks like the design | Matches Figma frame `Checkout / Promo v3` at 375px and 1440px |

---

## 6. Writing tickets as user flows

A lot of our work is naturally a sequence of steps, and many of our PMs already write
tickets this way. **Keep doing it.** A numbered flow shows *order*, which a flat checklist
can't, and it's usually the fastest way to describe a screen-by-screen change.

Flows and AC are not competitors. Use both:

| | Covers | Read by |
|---|---|---|
| **Flow** | The happy path, in order | Developer, to understand the shape of the work |
| **AC** | What must be true, including failures | QA, to decide pass or fail |

### Rules for flows

1. **One block per flow.** If a feature can be triggered two ways (an API call and a UI
   click, say), that's two flows — not one flow with branches buried in it.
2. **End every flow with the observable result.** "Status changes from Approved to
   Cancelled" is the payoff. A flow that ends at "user clicks Confirm" hasn't said what
   happened.
3. **Steps are user actions and system responses, not implementation.** "User selects a
   date and reason" — yes. "Add a date picker component" — no.
4. **Flows are the happy path. Failures go in AC.** This is the rule most often missed, and
   it's where the outsourced QA's questions come from.

### Worked example — a two-flow ticket

> **Title:** `Contract: cancel an approved contract`

```markdown
[Figma](https://figma.com/...)

## Background
Client needs to cancel contracts without waiting for an approval round. An approved
contract should be cancellable both from the UI and via API.

## Flow

### Flow 1 — via API
1. Client calls the cancellation API with contract number, cancellation date, and reason.
2. System sets the contract status to Cancelled. **No approval required.**

### Flow 2 — via e-Memo UI
1. User opens the Contracts menu and selects a contract with status Approved.
2. User clicks Cancel Contract.
3. User enters the cancellation date and reason, then confirms.
4. Contract status changes from Approved to Cancelled.

## Acceptance Criteria
- [ ] Both flows produce the same final status and are visible in the contract history.
- [ ] Cancelling without a date or a reason is rejected, in both flows.
- [ ] The cancellation reason appears in the document's Comment tab.
- [ ] Given a contract is already Cancelled, when either flow runs again, then it's
      rejected and the original cancellation is unchanged.
- [ ] Given the API references a contract number that doesn't exist, then it returns an
      error and no contract is modified.
- [ ] A user without cancel permission does not see the Cancel Contract button.

## Notes / Constraints
- Cancellation reason is free text, not a dropdown. (Confirmed with client 4 Aug.)
- The system calls the client's callback endpoint once a contract is cancelled.
```

Two things to notice.

**There's no "Items to implement" section.** The old habit would add *"add Cancel button,
add popup, make both fields required."* Every one of those is either already in Flow 2 or
restated as an outcome in AC. Deleting it removes a copy that could go stale.

**The AC lines are the ones nobody asks about upfront.** Already-cancelled contracts,
missing contract numbers, permissions. The flows are the easy part; these are what QA will
ask on the day, and answering them in the ticket is cheaper than answering them in a
comment thread three weeks later.

### Scope changes: edit the description, don't only comment

This is the most expensive habit to fix, and it has nothing to do with flows.

When a requirement changes after a ticket is open — a dropdown becomes free text, a
callback gets added — **update the description, then comment.** Not comment alone.

Why it matters, concretely: if the description still says "select a reason from a list"
while comment #7 says "free text", then

- a developer reading top-down builds the wrong thing,
- **QA testing against the description fails correct work**, and
- six months later nobody can tell which one won.

The comment is still useful as the audit trail of *when* and *why* it changed. It just
can't be the only place the new truth lives. Rule of thumb: **the description is the spec;
comments are the history.**

Two related habits:

- **New scope needs new AC.** If a change adds work (that callback endpoint), it needs its
  own AC line, and it's worth asking whether the estimate still holds.
- **An unanswered question in a comment is a blocked ticket.** If a developer asks
  something that changes what they build, add `Needs::Info` (§7.4) — don't leave the
  question sitting in a thread on a ticket that still looks `Status::Doing`. A question
  nobody has to answer is a question nobody answers.

---

## 7. Labels

Start with **four** label groups. Resist adding more until a group is genuinely missing;
every extra label is a decision a PM has to make on every ticket.

We use GitLab **scoped labels** (`Group::Value`) for the first three, because GitLab
enforces only one value per group — a ticket cannot be both `Status::In Progress` and
`Status::Done`.

### 7.1 `Type::` — what kind of work is this?

| Label | Meaning |
|---|---|
| `Type::Feature` | New implementation on a from-scratch project — nothing to change a request against yet. |
| `Type::CR` | Change Request. New or changed user-visible behavior on an existing product, billable. |
| `Type::Free Request` | Same as a CR, but not billed to the client. |
| `Type::Bug` | Something already delivered behaves wrong. |
| `Type::Chore` | Necessary work with no user-visible change (upgrades, cleanup, CI). |

Three rules keep this honest:

1. **A bug is never a Free Request.** If we delivered it broken, it's `Type::Bug`,
   regardless of who absorbs the cost. Reclassifying bugs as free features hides our
   real defect rate.
2. **`Type::CR` vs `Type::Free Request` is a billing decision, and the PM makes it.**
   Developers never change this label. The work is identical either way.
3. **`Type::Feature` is not a way to avoid the CR/Free Request billing call.** It's for
   work with no existing delivered product to change — a new project, a new app, ground
   that hasn't shipped before. The moment there's a shipped baseline being changed, it's
   a CR or Free Request, not a Feature.

Note we have no `Improvement`, `Enhancement`, or `Task`. Those all collapse into the
five above, and arguing about which is which produces no value.

### 7.2 `Status::` — where is it right now?

| # | Label | Meaning | Who moves it here |
|---|---|---|---|
| 1 | `Status::Backlog` | Captured. Not ready to work on — AC may be missing. | PM |
| 2 | `Status::To Do` | **AC written, dev can start.** Committed work. | PM |
| 3 | `Status::Doing` | A developer is actively on it. | Developer |
| 4 | `Status::Ready to Merge` | MR open and approved, awaiting merge. | Developer |
| 5 | `Status::Deployed to Development` | Merged and on the dev environment. Nobody is testing it yet. | Developer |
| 6 | `Status::Testing` | Someone is actively verifying it against the AC. | QA / Developer |
| 7 | `Status::Testing Passed` | Verified on dev. Waiting for the UAT deploy. | QA |
| 8 | `Status::Deployed to UAT` | On the UAT environment for client review. | Developer |
| 9 | `Status::Ready to Release` | Signed off. Waiting for the release window. | PM / QA |

Three rules that make this worth having:

1. **Nothing enters `Status::To Do` without Acceptance Criteria.** This is the gate, and
   the single most important rule in this document. `Backlog` is where tickets live until
   their AC is written.
2. **The person doing the work moves the label.** A PM chasing status is a PM not doing
   their actual job.
3. **A ticket sits in exactly one Status.** GitLab's scoped labels enforce this.

**On `Testing` (6) vs `Deployed to Development` (5):** these are different states, not
duplicates. `Deployed to Development` means the code is deployed but unverified —
tickets can sit here for days. `Testing` means someone is looking at it right now.
Testing usually happens on the dev environment, occasionally on UAT; the label describes
*who is doing what*, not which server it's on.

**On `Testing Passed` (7):** this is a result, not an activity — nobody is working on a
ticket sitting here. It exists so that "QA is happy with this" and "the client can see
it" stay separate facts, which matters because the UAT deploy usually happens in batches
rather than per ticket. Two things follow:

- **QA moves it, not the developer.** It's QA's verdict.
- **Watch the queue.** A pile-up here means UAT deploys aren't keeping pace with testing,
  which is a release-cadence problem rather than a ticket problem. Worth a look in retro.

#### When QA or the client rejects it → back to `Status::Doing`

A rejection means the work is unfinished, not that it's new. It was already prioritized
once, and on a team this size the developer who wrote it still has the context. So it
goes straight back to `Doing` — not to `Backlog`, not to `To Do`.

Two consequences, both intentional:

- **The ticket's cycle time now includes the rework.** It will look slow. That's the
  point: rework you can't see is rework you can't reduce.
- **Add a comment saying what failed.** The AC checkboxes that failed get unticked. A
  ticket bouncing back with no written reason will bounce again.

**Exception:** if the rejection reveals the *AC itself was wrong* — we built what the
ticket asked for and the ticket asked for the wrong thing — leave the Status where it is
and add `Needs::Discussion`. That's a specification problem, not a code problem, and it
should look different on the board because a different person has to fix it.

#### No `Blocked` status

If a ticket can't move forward, leave the Status as-is and add a `Needs::` label (§7.4).
You want to see *where* it stalled, not just *that* it did — a ticket stuck at
`Deployed to UAT` and one stuck at `To Do` are very different problems.

### 7.3 `Priority::` — what do I pick up next?

P-numbers only mean something if each one carries a written expectation. These are ours:

| Label | Meaning | Expectation |
|---|---|---|
| `Priority::P0` | **Production is down or unusable for real users.** | Drop everything. Same day. |
| `Priority::P1` | Needed for the next release, or a hard client deadline is at risk. | Next up, ahead of P2. |
| `Priority::P2` | Default for all normal work. | Normal queue order. |
| `Priority::P3` | Nice to have. No date attached. | When there's slack. |

`P2` is the default — if a PM hasn't thought about priority, the ticket is P2, and a
board that's mostly P2 is a healthy board.

Two guardrails:

- **P0 is for production, not for pressure.** A client asking loudly is not a P0. If
  everything urgent becomes P0, we lose the ability to signal an actual outage.
- **If more than ~20% of open tickets are P0 or P1, the labels have stopped meaning
  anything.** The PM owns keeping this honest, and it's worth checking at every
  retrospective.

### 7.4 `Needs::` — flags (unscoped, may combine)

| Label | Meaning |
|---|---|
| `Needs::Info` | Blocked on an answer from the PM or client |
| `Needs::Design` | Blocked on design assets |
| `Needs::Discussion` | Approach isn't obvious; needs a short conversation |
| `Needs::Estimation` | Blocked on a man-day estimate before it can be committed to a release |

These are the escape hatch. When a developer finds the ticket under-specified, they add
a `Needs::` label and a comment saying exactly what's missing — they do not guess, and
they do not silently stall.

### 7.5 Not labels

Use GitLab's built-in fields instead — they're searchable, sortable, and free:

- **Who** → assignee
- **Which release** → milestone
- **Which project area** → the title's `<Area>` prefix, or a `Component::` group later
  if a project genuinely needs it

---

## 8. Definition of Ready / Definition of Done

Two short checklists. These are the convention's teeth.

### Ready — `Backlog` → `To Do` (PM → developer)
A ticket may be moved to `Status::To Do` only when:

- [ ] Title follows `<Area>: <outcome>`
- [ ] Background is filled in — why now, and the outcome (1-2 lines)
- [ ] Acceptance Criteria are written, and each line is independently checkable
- [ ] `Type::` and `Priority::` are set
- [ ] Designs / API contracts are linked if the work needs them

That's it — five items, and only the AC line takes real thought.

If a developer picks up a ticket that fails this, they add `Needs::Info` and comment.
That's not obstruction; it's the process working.

### Done — `Ready to Release`
A ticket may be moved to `Status::Ready to Release` only when:

- [ ] Every AC checkbox is ticked
- [ ] MR merged and code-reviewed
- [ ] Verified against the AC by QA (this is why AC must be readable by someone who
      didn't write the code)
- [ ] Client / PM sign-off obtained if the ticket needed UAT

Note `Ready to Release` is the last status in this convention — it means *done and
waiting for the release window*. Tracking what actually shipped is release management,
not ticket hygiene; use GitLab **milestones** for that.

---

## 9. Bugs

Bugs use the same labels but a different body. `Type::Bug` + this template:

```markdown
## What happens
<!-- Observed behavior. -->

## What should happen
<!-- Expected behavior. -->

## Steps to reproduce
1.
2.
3.

## Environment
<!-- App version, device / browser, environment (prod / staging), user account or role. -->

## Evidence
<!-- Screenshot, screen recording, log excerpt, or request ID. -->
```

A bug with no reproduction steps gets `Needs::Info`, not a developer.

The "What should happen" section *is* the acceptance criteria for a bug — no separate
AC section needed. Bugs still pass the same `Backlog` → `To Do` gate: a bug is Ready
when it has reproduction steps and a clear "what should happen".

---

## 10. Rolling this out

Suggested order, smallest change first:

**This is not a blank slate.** Projects already have partly-migrated labels — for example
memo-tss currently runs `backlog` (lowercase, unscoped) alongside `Status::Done`, plus
`High`, `Priority2`, and `Pending Client` all doing priority's job, and
`Pending deploy on PROD` acting as a status outside the `Status::` group. Step 0 is
cleaning that up, or the new taxonomy just becomes another layer on the pile.

0. **Week 0 — audit and consolidate.** Per project, list labels in use and map each one to
   its new home (`backlog` → `Status::Backlog`, `High`/`Priority2` → `Priority::P1`/`P2`,
   `Pending deploy on PROD` → `Status::Ready to Release`). Delete the old label once
   remapped, so there's exactly one way to express each thing. Labels like `Backend`,
   `Frontend`, and client names (`Peerapat`) are a different axis and can stay — they're
   effectively `Component::`.
1. **Week 1** — Add the description templates to GitLab (`.gitlab/issue_templates/CR.md`
   and `Bug.md` in each repo's default branch; they then appear in the description
   dropdown). Create the label set at **group level**, not per project, so every repo
   shares one taxonomy. Nothing else changes yet.
2. **Week 2** — All *new* tickets use the template. Existing tickets are left alone; do
   not backfill.
3. **Week 3** — Turn on the Ready gate: developers stop starting tickets that aren't
   `Status::To Do` with AC. Expect friction here. This is the step that actually changes
   behavior, and the step most likely to be quietly abandoned.
4. **Week 4** — Retrospective on the convention itself. What did PMs find painful to
   fill in? Which labels went unused? Cut what isn't earning its place.

Three things to watch for:

- **The template becomes decoration.** Watch the *required* sections specifically: a Goal
  restating the title, or AC that just says "works as expected", means the Ready gate isn't
  being enforced. (Empty *optional* sections aren't a problem — those should just be
  deleted.)
- **Label sprawl.** Restrict label *creation* to a small group in GitLab, and revisit
  the taxonomy quarterly.
- **`Backlog` becomes a graveyard.** The AC gate means tickets pile up in `Backlog`
  waiting on the PM. That's the gate working — but it makes the PM the bottleneck, so
  the backlog size is now a number worth watching in retro.

---

## 11. Quick reference

**Flow:**

```
Backlog → To Do → Doing → Ready to Merge → Deployed to Development
                    ↑                                    ↓
                    │                                 Testing
                    └────────── rejected ─────────────────┤
                                                          ↓
                                       Testing Passed → Deployed to UAT
                                                                ↓
                                                       Ready to Release
```

| | |
|---|---|
| **Required** | Background (1-2 lines) · Acceptance Criteria |
| **Optional** | Flow · Out of Scope · Notes / Constraints — delete if unused |
| **AC format** | None required. Checkable by QA is the only test. |
| **Every ticket carries** | one `Type::`, one `Status::`, one `Priority::` |
| **Types** | `Feature` · `CR` · `Free Request` · `Bug` · `Chore` |
| **Priorities** | `P0` prod down · `P1` next release · `P2` default · `P3` nice to have |
| **Flows** | Encouraged. Happy path in order; failures go in AC. |
| **The gate** | no AC → stays in `Backlog` → nobody starts it |
| **Scope change** | edit the description, *then* comment. Never comment only. |
| **Rejected** | back to `Doing`, with a comment saying what failed |
| **Stuck** | keep the Status, add a `Needs::` label, say what's missing |
| **Never** | a bug relabelled as `Free Request` |
