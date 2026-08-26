---
name: upward-comms
description: Rewrite engineer-to-engineer content for engineering-org leadership (VPs, directors, PMs, release managers, execs at an engineering-savvy company) and shape it for the channel it is going to: ticket comment, chat post, async standup line, email, or meeting talking-points. Use when the user asks to write/rewrite for management, exec, VP, director, PM, or release manager; asks for an "executive summary", "leadership update", or "status update"; says "make this less technical" or "less jargony"; or asks for a chat, slack, teams, email, standup, or meeting version of work originally written engineer-to-engineer.
---

# Upward Comms

Translate engineering content for engineering-org leadership and **shape it for the channel** — ticket comment, chat post, async standup, email, or meeting talking-points. The audience reads code names but not code. The channel decides the length, formatting, and how much structure to leave on the page.

This skill is **draft-only**: it produces text you copy and paste into the destination yourself. It never posts to a tracker, chat tool, email, or any other channel.

If the channel isn't stated, ask one short question — *"ticket, chat, standup, or email?"* — and stop.

## Audience — what "engineering-org leadership" means

Engineering-savvy non-engineers: VPs, directors, PMs, release managers, execs in companies that ship technical products. They read product/framework names and cross-reference ticket keys and PRs. They do not read code.

They want: *what's the state, what does it mean for customers, who owns it, what's next.* They do not want: how the bug works at the function level.

This is **not** for marketing, finance, customer-facing, or true ELI5 audiences — those need a different rewrite. Flag and confirm before producing one.

## Tone

**Keep.** Product names, service/feature names, team-owned component names, ticket keys, PR/MR numbers, customer/workflow identifiers (`Checkout`, `Redis`, `Postgres`, `the kiosk flow`, `#123`, `MR !456`). These are the bridge between engineering and leadership tracking.

**Strip.** Function names, file paths, struct/object fields, commit SHAs, code expressions, env var names, line numbers, internal data-structure jargon (`getCart`, `cart.service.ts`, `req.session.cartId`, `cart:<cartId>`, `0e0a6bac`). None of this is actionable to the audience.

**Translate.** Mechanism into one or two sentences of plain-English cause-and-effect. Not *"`getCart` builds the Redis key from `req.session.cartId`"* but *"the cart cache was keyed off the browser session instead of the user, so a shared browser served the previous person's cart."* Translate without lying — a race stays a race; a regression stays a regression.

**Don't over-strip.** Engineering-org leadership reads concept-level technical vocabulary fluently — *race condition, cache, session, TTL, regression, rollback, migration, queue, endpoint, timeout*. The line is between *concept exists and matters here* (keep) and *here's the function/field/file/SHA* (strip). Replacing "race condition" with "timing issue" patronizes the reader.

**Bias toward** active voice, concrete subjects, short paragraphs. *"We found the bug. Alex wrote the fix. PR is up for review."* beats *"The root cause has been identified and a fix has been authored and submitted for review."*

**Avoid:**

- Hedging that isn't really hedging (*"we believe," "appears to," "may have"*). State it or don't.
- Re-stating the obvious for thoroughness (*"This bug is in Checkout, which handles carts, which is important for revenue, which..."*).
- Telling leadership how to do their job (*"you should prioritize," "this needs to land before X"*). Give them the facts; they decide.
- Engineering-process minutiae: bisect runs, debug iterations, GDB sessions. They care that you found it, not how. (Exception: when the *process* itself is the story — *"we burned three weeks before realising the bisect was misleading"* — then a single sentence as a learning, not a play-by-play.)

## Channel shapes

Same content, different shell. Pick the shape that matches where it's going.

### Ticket comment / written status report

Full structured block. Bolded section labels. Easy to scan from the ticket page.

Building blocks (use as many as fit):

- **Status / TL;DR.** One bolded line. Reader can stop here and have the right answer. *"Fixed pending merge."* / *"Root cause unknown — investigating."* / *"Blocked on vendor."* / *"Customer-visible regression in production; rollback in flight."*
- **Impact.** Who's affected, how badly, what they see. Customer / workflow / product terms, not test-suite terms. *"Checkout fails for everyone using PayPal"* > *"the test fails."*
- **What broke.** Short paragraph. Plain-English mechanism, one level of why, no code identifiers.
- **Why now / how it slipped through.** Optional. Include when leadership will ask anyway: latent regression, CI gap, prior incomplete fix, change that landed during a freeze.
- **Owner.** Person + team + their PR/branch/ticket artifact. One link, not five.
- **Next steps.** Concrete, near-term, ordered. *"Code review → merge → deploy."*
- **Workaround / mitigation.** If customers are hitting it now, what can they do today? One sentence.
- **Risk.** Optional. Real risks only — *"fix touches the hot path; perf regression possible until benchmarked."* Don't manufacture risk to look thorough.

Order by what matters most for *this* item.

### Chat — channel post or DM

Single message, no walls of text. Heavy bolded section labels read as "I escaped from a ticket" — don't.

- One **bolded TL;DR** as the first line.
- 2–4 short bullets underneath: impact, owner+link, next step. Drop blocks that don't apply.
- One link, embedded inline (`#123` / `MR !456`). Not a link wall.
- No greeting, no signoff. The channel is the context.
- If it's a **thread reply** rather than a new post, lose the TL;DR — just lead with the answer.

Length target: under ~80 words for a top-level post; under ~40 for a thread reply.

### Async standup note

The audience scans 10 of these in 30 seconds. Front-load the verb.

- 1–3 lines, max.
- Pattern: *"\<state\> \<thing\>. \<owner if not me\>. \<next\>."*
- Examples:
  - *"Fixed cross-user cart leak on shared devices (#123). Maya's MR !456 in review. Deploy flushes stale keys."*
  - *"Still chasing the intermittent checkout timeout. Reproducer is reliable now; narrowing it down. No ETA yet."*
- No bullets, no bolded labels. The format **is** the sentence.

### Email — internal exec / cross-team

Subject line is half the value.

- **Subject:** the TL;DR rewritten as a noun phrase. *"Cross-user cart leak on shared devices: fix in review (#123)."*
- **Greeting:** match the recipient register (*Hi Sam,* / *Hi all,*).
- **Body:** the ticket-comment shape, but as flowing paragraphs separated by blank lines rather than bolded section labels. Two or three paragraphs is plenty.
- **Sign off** with the next decision point that needs the recipient's attention, if any. If none, a plain *"— [Name]"* is fine.

### Meeting talking-points

You're going to *say* this, not show it.

- Bullet list, max one short clause per bullet.
- Order is the order you'll speak in.
- Include the numbers/keys you want to reference out loud, in the bullet itself, so you don't fumble.
- Skip prose. *"Customers on shared devices saw the wrong cart."* / *"Root cause: cart cache keyed by session, not user."* / *"Maya's fix in review, MR !456."* / *"Deploy flushes stale keys once it lands."*

## Source material

The input is one of:

1. **Pasted technical text** → use directly.
2. **A ticket/issue key or link** → if the user has a tracker tool available (e.g. an MCP for their issue tracker), fetch the current state and reframe the most recent substantive comment — don't dump the full thread. If no such tool is available, ask the user to paste the relevant content.
3. **The current conversation** → if you (or the user) just produced engineering content and the user now says *"now in chat"* / *"now for the VP,"* reuse what's in context.

If the source is ambiguous, ask one question and stop.

## Output flow

1. **Produce the draft** as a single chat block, formatted as the channel would render it. Hand it off for the user to copy — never post it anywhere.
2. **One iteration is normal, three is a smell.** If the user is on the third revision, ask what specific framing/audience assumption you're missing — don't keep tweaking blindly.

## Worked example — same bug, three channels

**Source (engineering issue comment):**

> **Mechanism:** `getCart(userId)` in `cart.service.ts` builds the Redis key from `req.session.cartId` instead of `userId`. `cartId` is only regenerated on logout, not on login, so when two users share a browser (kiosk / shared device) the second login reuses the first session's `cartId`. Cache hit on `cart:<cartId>` returns the previous user's line items. TTL is 24h so it persists. Repro: log in as A, add items, log out, log in as B → B sees A's cart. Only happens on the read path; checkout's `POST /orders` re-reads from Postgres so orders themselves are correct.

### As an issue comment

> **Status: Root cause found, fix in review.** MR up, validated against the repro.
>
> **Impact:** On shared devices (kiosks, family computers), a customer who logs in right after someone else could briefly see the previous person's shopping cart — their items, not their payment or address details. Carts only, and only on the page view; placed orders were always correct. Low volume but a customer-trust and privacy concern.
>
> **What broke:** The cart cache was keyed off the browser session rather than the logged-in user. On a shared browser the new login inherited the old session's cache entry, so the lookup returned the previous user's cart. The entry stuck around for 24 hours, which is why it was visible well after the first user left.
>
> **Why now:** The session-keyed cache has been there since launch, but it only surfaces when two accounts use the same browser back-to-back — rare enough that it slipped past QA and only showed up once we had kiosk customers.
>
> **Owner:** Maya (Checkout team). MR !456.
>
> **Next steps:** review → merge → deploy. We're also flushing the affected cache keys on release so no stale carts survive the fix.

### As a chat post

> **Cross-user cart leak on shared devices — root cause found, fix in review.** (#123)
>
> - Cart cache was keyed by browser session, not user → on a shared browser the next login saw the previous user's cart items. Carts only; orders/payment unaffected.
> - Owner: Maya, MR !456 in review.
> - Deploy will flush stale cache keys so nothing lingers.

### As a standup note

> Found the cross-user cart leak (#123) — cart cache keyed by session instead of user. Maya's MR !456 in review; deploy flushes stale keys.

What changed between channels: same diagnosis, same owner, same next step. The issue comment gets every block. Chat drops "why now" — too much for the channel. Standup keeps just state + key + owner + next. None of them mention `getCart`, `cart.service.ts`, or `req.session.cartId`.

## Rules

- **Never invent facts** to make the rewrite cleaner. If the engineering source says "root cause unknown," the rewrite says "root cause unknown" — do not promote a speculation to a finding for narrative tidiness.
- **Never strip a ticket key, PR number, or customer/workload name** during de-jargoning. They're the cross-reference bridge — losing them breaks tracking.
- **Never invent owners.** If the source doesn't name one, ask the user — don't guess from `git blame` or recent commits.
- **Stay out of advocacy.** This skill produces a status update, not a recommendation. If the user wants a recommendation memo, confirm before reframing.
