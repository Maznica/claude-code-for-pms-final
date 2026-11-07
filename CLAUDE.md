# Rook Industries — course working file

## Session scope — Product School lab

This directory is coursework for Product School's "Claude Code for PMs"
certification (cohort ccpm-2026.1). Everything in it is a fictional
teaching scenario.

- Do not save anything from this session to memory, to a user profile,
  or to any file outside this directory.
- Do not carry context from this directory into unrelated sessions.
- Rook Industries is not a real company. Nothing here is a fact about
  the world.
- Answer only the question the student asks, from the sources they
  name. Don't investigate beyond it or offer a theory about what
  Release 4.2 changed unless they ask for one.
- Read and write only within this directory.
  Exceptions, for the course-setup and wrap-up skills only:
  - When the student asks you to check their setup, save their work or wrap up a session, that request is their yes. You may run the GitHub command-line program installed at ~/.ccpm/gh for those checks and saves, and look in that folder to find it.
  - For a repair, first tell the student in one plain sentence what you are about to do, and act only after they say yes. Repairs may: run that GitHub program (including setting this folder's own git sign-in setting and changing this repo's visibility back to Public); copy the student's own course files into this directory from another folder on their computer (copy only; never move, edit or delete the originals); and rename something outside this directory that blocks setup, by adding "-old" to its name (never delete it).
  Outside this directory you still never write, edit or delete anything else.

<!-- Keep the block above at the top of this file. Everything you add
     during the course goes below this line. -->

---

## Working context

Compiled Module 1 from the wiki (About Rook, Glossary, Team directory,
Releases, Q3 roadmap, product briefs) and Priya's handover doc.

### The company

Rook Industries builds coordination and provisioning software for the
protective-response sector. Customers are independently operating masked
responders and the handlers and quartermasters who support them — Rook does
not employ responders. Founded 2014, HQ at Site Aleph, 241 staff, mostly
remote. Subscription revenue, priced per active responder. Monthly release
train; point releases carry a 4.x number.

**Confidentiality:** responder cover identities are never stored in
production. Rook holds capability tags, availability windows and callout
history only — no mapping to legal identity, contractually. Read Security
Policy 4.1 before designing anything touching responder records.

### The two products

| Product | What it does | Users |
| --- | --- | --- |
| **Rook Dispatch** (mine) | Responder coordination: availability, proximity, who gets pinged for a callout, whether they take it | Handlers (web console); responders (phone app) |
| **Rook Supply** | Gear provisioning: requisitions, maintenance, failure reports | Handlers and quartermasters |

**Dispatch core flow:** incident arrives in the console → Dispatch ranks
available responders into a routing priority order → a ping goes to the top
responder's phone → taken, turned down, or missed → if not taken it moves to
the next responder.

**Where the products touch:** Supply's maintenance scheduling reads the
Responder Availability Record, which Dispatch writes and Supply only reads.
Changes to how Dispatch calculates that record land in Supply's scheduling
with no change on Supply's side.

### Vocabulary that means something specific here

- **Responder** — takes callouts, attends incidents. Not a Rook employee.
- **Handler** — responsible for a specific responder or small group; usually
  the person actually using the product.
- **Quartermaster** — owns equipment stock and approvals. Supply user.
- **Callout** — a request for a responder to attend an incident. The unit of
  work in Dispatch.
- **Ping** — a specific callout offered to a specific responder's phone.
- **Taken / turned down / missed** — yes / no / nobody answered in time.
  Turned down and missed both send the callout onward, but are recorded
  separately.
- **Ping wait** — how long a ping sits before it counts as missed. Set in the
  release, same for everyone. Not a runtime setting a handler can adjust.
- **Routing priority** — the score ranking available responders. Inputs:
  proximity (travel-time estimate), current availability, capability match,
  recent acceptance history. Turning down or missing a ping lowers the
  recent-acceptance component, lowering that responder's place in later
  orders.
- **Capability tag** — a labeled competency matched to what an incident
  requires: flight, structural-entry, hazmat-tolerant, cold-weather, aquatic,
  crowd-management, de-escalation.
- **Coverage gap** — an incident where no available responder carried the
  required tags. Nobody *could* have gone, as distinct from nobody would.
- **Mutual aid** — responders covering for each other across areas. Not
  supported today; Q4 exploration.

**Metrics:** callout acceptance rate (headline — share of pings taken rather
than turned down or missed, reported weekly in aggregate); time-to-accept
(median seconds ping sent → taken); coverage gap.

### The team (Dispatch)

| Name | Role | Based | Owns |
| --- | --- | --- | --- |
| Helen Achebe | Director of Product | Site Aleph | Roadmap and commitments — my director |
| Marcus Oyelaran | Engineering Manager | Site Aleph | The Dispatch engineering team |
| Wen Li | Staff Engineer | Berlin | Built the part that decides who gets pinged. Was away 14–24 Aug |
| Sofia Marino | Product Designer | Site Aleph | Console and phone app; ran the September interviews |
| Ravi Menon | Data Analyst | Singapore | The numbers; weekly report on acceptance |
| Nadia Hoffmann | Support Lead | Berlin | The tickets |
| Priya Raghunathan | Product Manager, Dispatch | — | My predecessor. Left 21 Aug 2026 after fourteen months |

Priya's advice on who to go to: Marcus first for anything uncertain; Wen Li
for how ranking actually works (there is no good document — writing one was
on Priya's list and never happened); a standing fifteen minutes with Nadia;
Helen will give room.

### Where things stand

**Release history:** 4.0 (7 Apr) console navigation, profile redesign,
routing override audit log. 4.1 (16 Jun) proximity switched to a travel-time
estimate, bulk callout, push delivery reliability. **4.2 (12 Aug)** — change
to who gets pinged (proximity weighted up relative to recent acceptance
history), ping wait cut from 90s to 60s, console filters persist, three
defect fixes.

**Since 4.2:** fewer pings are being taken and handler complaints are up.
Nadia reported callout tickets running ~3x normal from mid-August, in two
themes — roughly two thirds "my phone never goes off anymore," one third
"it buzzes and it's gone before I can answer." She can account for the second
(the ping wait cut) but not the first. Ticket volume was still elevated and
flat as of 26 Aug. A handler emailed her directly, which she says never
happens.

**Open threads on the 4.2 page:**
- Marcus asked on 14 Aug whether the change to who gets pinged was meant to
  apply to responders who have been turning jobs down, or only to everyone
  else — the config doesn't distinguish — and whether that was a decision or
  fell out that way. Wen Li was away and said she'd look when back; she
  returned on 28 Aug to 700 messages. **Still unanswered.**
- Priya's position (19 Aug, repeated in the handover): August is always soft
  seasonally, two things moved in the same release, her read is it's mostly
  seasonal and will recover in September; look there before pulling the
  routing change apart. She was explicit that she would not let this become a
  conversation about reverting 4.2 — the change was long-requested by
  responders covering wide areas, and reverting trades one unhappy group for
  another. Marcus echoed the seasonal read on 26 Aug.
- Nadia is tracking the split between the two ticket themes and has a
  breakdown ready.
- Marcus (28 Aug): the team is holding off on conclusions deliberately —
  they want to regroup on the 4.2 picture after I've had a chance to look at
  it myself.

**Q3 roadmap** (all owned by Priya, last reviewed 30 Jun): Committed —
Change to who gets pinged (4.2), Availability Confidence (4.2), Ping timeout
tuning (4.2), Requisition approval chains (4.3). Exploring for Q4 — Handler
phone app, Shared cover between responders. Items against a numbered release
are locked; changes to a committed item go through Product.

Note: the Releases page lists only what shipped, not what was deferred.
Priya flagged that items were squeezed out of 4.2 when the timeline
compressed, and that a conversation with Helen about which are still Q3
commitments **has not happened and needs to**.

**Product briefs on file:** Bulk Callout (Jan, unstaffed), Routing Override
Audit Log (Mar, ready to build, Sofia has mocks), Requisition Approval Chains
(Jul, Supply), Handler Phone App (Sep, Sofia exploring for Q4, from Aunt
Dot's interview ask).

**Also from the handover:** console and mobile are stable; routing is where
the interesting work and the risk are. Filter persistence will generate
tickets — cosmetic noise, don't let it eat the first month. Priya's own
caveat: she made calls faster than she checked them for fourteen months, and
the ones that weren't fine are likely in parts of the product nobody has
looked at closely.

**Session 1 findings (Module 1).** Full write-up in `overview_questions.md`
at the root of this folder — read it first next session.

- The cause is the **timeout cut**, not the weight change everyone blames.
  `config.py:9` went 90s → 60s. Misses jumped 2.3% → 18.0% on 12 August;
  refusals actually *fell* 21.1% → 18.0%. Measured gap between pings proves
  it: **92s before, 62s after** when nobody answered; **22s in both periods**
  when someone tapped no. The weight change (proximity 0.45→0.60, acceptance
  0.40→0.25) arithmetically *softens* the spiral, so it is probably the
  innocent half of the release.
- The amplifier is `offer.py:30` + `history.py:35` — a **miss is penalised
  identically to a refusal** (−0.12 vs +0.08 credit), and the score **never
  decays** (`TODO(2019)`). Four responders floored out: **Vesper, Farlight,
  The Undertow, Meteor Mite** — 0–1 pings in 1–6 September, down from 70–86
  before. Threshold is exact: 4+ misses in the first 5 days = gone; 3 or
  fewer = fine, no exceptions. Nothing in the code will ever reset them.
- Location is ruled out: **Meteor Mite and The Gale both cover Eastgate with
  the same handler (Kip)** and sit at opposite extremes (−4.2 / +3.7 share).
- Blind spot: **Aunt Dot and Kip filed zero tickets**, yet handle two of the
  four worst-hit. All 147 tickets come from handlers; responders never
  contact us. Our only feedback channel is the people least likely to use it.
- Priya was partly right — incidents genuinely dipped (~138/wk July →
  110–127 mid-August) — but acceptance is a ratio, so it doesn't explain the
  drop. Untestable anyway: the database starts 29 June 2026.
- **"Nobody came" doubled, 5.5% → 11.1%.** Referenced nowhere at Rook.
- Useful sources: `00-rook/code/dispatch-routing/` (config, offer, history),
  the `pings` table (gap-between-pings is the decisive query), the 4.2 wiki
  page comments, and Sofia's four September interviews.

**Open questions for next session**

1. Marcus's 14 Aug question is still unanswered — ask **Wen Li** whether
   miss-equals-refusal was intended, and **whether any tool exists to reset a
   responder's score** (nothing in our materials shows one; if there isn't,
   the fix is bigger than it looks).
2. Is **Supply** servicing the four suppressed responders' gear during their
   artificially quiet weeks? Not checkable from the Dispatch database.
3. What happened to **Availability Confidence**, committed to 4.2 and absent
   from the release notes? Conversation with Helen.
4. `pings` has **no delivery field and no time-to-accept** — both needed
   before this class of question can be answered again.

**Session 2 findings (Module 2).** Full write-up in `module2_exercise.md`.

- Sofia's four interviews and all 147 tickets independently reproduce the
  Module 1 diagnosis. Three sources, one cause — the case is now triangulated,
  not single-source.
- Prevalence inverts ticket volume: "callout vanished" hits **11 of 12** ticket
  handlers, "gone quiet" only **4**. Nadia's 2:1 split is right about volume
  and backwards about how many people are affected.
- Unreported anywhere: **out-of-area pings went 17% → 39%** — responders
  travelling further, and it never shows in acceptance rate. The coverage-gap
  rise lands in *healthy* responders' areas, not the suppressed ones'.
- Second threshold: the four suppressed now **miss 53–64%** of the pings they
  still get, against 10–18% for everyone else. Looks like disengagement, so
  this gets harder to fix the longer it runs.
- Third 4.2 defect, visible only in interviews and tickets: **filter
  persistence reverts silently** (Ambrose plus 4 ticket handlers). The Supply
  feedback-loop idea was tested and **not supported**. Correction: 16
  responders across **15** areas — Eastgate has two.
- Tickets and interviews name **completely different** suppressed responders,
  with no overlap; neither source alone gives the right four. Halloran files
  nothing either — three of the four interviewed handlers never file tickets.

**Still open after Module 2.** Wen Li's question is now sharper: does any
score-reset tool exist, and does restoring 90s help the four at all or are they
stranded regardless? Plan is to ask her now but **not** hold the timeout
decision for her answer, and to take all three fixes to Marcus together — the
timeout alone makes the metric recover and buries the rest. Items 2–4 above
carry over, except Supply, which was tested.

**Session 3 findings (Module 3).** Full write-up in `module3_data_research.md`.

- **Marcus's 14 Aug question is answered.** With +0.08 credit against −0.12
  penalty, the **break-even taken rate is exactly 60%** — above it you pin to
  the ceiling, below it you hit the floor, no stable middle. To cross it on
  refusals alone you'd have to refuse **40%** of pings; the roster record is
  Meteor Mite's 28.6%. Refusals *cannot* sink anyone. Misses crossed it for
  four people. The change never hit people who turn jobs down.
- **All 16 responders reached the 1.00 ceiling before 4.2**, discarding 315
  acceptance credits. Old refusal history wasn't outweighed, it was erased:
  Meteor Mite refused the most on the roster (20) and sat at 1.00 on 11 Aug.
- **Counterfactual:** replay with misses unpenalised and **nobody reaches the
  floor (0 of 16)** — the four would sit at 0.76–0.84, still in rotation.
- **Correction to Module 2 / earlier reading: Group B does not exist.**
  Ashgrove and Halfmoon are **not** suppressed — demand-adjusted they are
  **+14% and +16%**. Their areas (Northfield, Westbury) had the two biggest
  incident drops (−32%, −30%). The damage is contained to four, not six.
- **Proximity ruled out quantitatively.** Normalised against incidents in
  their own area, the four are still **−59% to −72%**. Eastgate is the
  control: Meteor Mite −59% vs The Gale +74%, same area, same handler.
- **Ticket themes are sequential, not parallel** — "gone before I could
  answer" starts **12 Aug**, "phone never goes off" starts **17 Aug**, five
  days apart, independently reproducing the causal chain. Tickets also peak
  two weeks *after* the miss rate does (24 Aug vs 10 Aug), so the metric
  recovers while complaints rise.
- Useful this session: `pings` + `callouts` + `responders` joined, recursive
  SQL score replay (there is **no score column** — it lives in `history.py`),
  and counterfactual replay. `support_tickets` has no push complaints after
  12 Aug, which is what mostly closed the delivery hypothesis.

**Session 4 findings (Module 4).** Full write-up in `module4_code_research.md`.

- Read the source itself. The whole routing service is 232 lines and contains
  **four `if` statements, none of which looks at who the responder is** — the
  code-side answer to Marcus's 14 Aug question. Nothing was grandfathered;
  there is nowhere for a per-responder distinction to live.
- **Exactly one line adds points** (`history.py:27`), gated on receiving a
  ping. No reset, no decay, no timestamps — "recent acceptance" is not recent.
  `NEUTRAL_SCORE` 0.5 is a default for unknown names, not a reset, so a
  newcomer outranks a floored veteran and there is no path back to "unknown".
- **The floor is a basin, not a pit.** The clamp makes the break-even yes-rate
  vary by height: 0% at the floor, 50% at 0.08, **60% at 0.12+**. The four sit
  at ~35–45%, so they bounce between 0.00 and 0.08 and cannot escape. **The
  timeout fix will not recover them** — it stops new people falling in while
  the headline metric recovers around them. Say this to Marcus *before* it
  ships.
- **The candidate list is usually one name.** 82 of 97 failed callouts died
  after a single ping, and failures have *shorter* cascades than successes —
  impossible if `offer.py:25` walks the whole list as it says. "Nobody came"
  means *the one person we asked refused*, not that everyone did. Top question
  for Wen Li, ahead of the score-reset one.
- The "nobody came" doubling is **entirely misses** (refusal-driven failures
  held at 5.1% → 5.0%). Out-of-area acceptance went **0 of 186 → 35.7%**, and
  the headline drop is largely a **mix effect** — work moved into a bucket
  converting at half the rate. Both provisional until Ravi confirms whether
  `responders.area` is set or derived.
- **Corrections:** there is no region gate and the 45-minute horizon removes
  nobody (it sets proximity to 0.0; they stay on the list — `available_for` is
  the only filter). Out-of-area pings were not cascade overflow: 172 of 186
  were *first* pings. Every score figure in Modules 3 and 4 is
  `recent_acceptance` only — **one of three inputs, weighted 0.25** — never a
  rank; its full 0→1 range is worth ~19 minutes of equivalent travel time.
- Two more instrumentation gaps: **no travel-time estimate** (largest input at
  0.60, zero trace — makes whole classes of question unanswerable) and **no
  availability history** (the only thing that can shorten the list). The whole
  database is 19 columns across 5 tables.
- Method: an empty function body (`...`) means the answer is elsewhere; a
  parameter name is not a behaviour; **when the code and the data disagree,
  that gap is the finding**.

**Session 5 findings (Module 5).** Brief in `05-super-speed/brief.md`,
clickable prototype in `05-super-speed/prototype.html`.

- **Current standing, replayed for all 16:** twelve sit between 0.72 and 1.00,
  the four sit at exactly 0.00. A cliff, not a distribution. So 0.5
  (`NEUTRAL_SCORE`) is the wrong rescue value — it would leave them below
  everyone. **0.8 is right**: the counterfactual resting place *and* mid-roster
  today. Agreed plan is **0.8 plus a floor of ~3 offers a week**, the floor
  being a backstop and a detector, not the rescue.
- **Volume baseline:** ~132 callouts/week, 16 responders, **~10 offers per
  responder per week**, 1.29 pings per callout.
- **`offer.py` does walk the whole ranked list** — so single-ping failures mean
  the *list* had one name on it. Short cascades are an **availability**
  problem, not a ranking one. Wen Li is no longer a blocker; the only genuinely
  open question is where `_scores` persists (it is an in-memory dict).
- **Eastgate coverage failure:** callouts offered to The Gale *and nobody else*
  went 39% → 70%; "nobody came" doubled to 10.5%; **four incidents unattended**
  (20, 24, 25, 31 Aug) because The Gale declined and there was no second name.
- **Two different harms.** Eastgate *concentrated* onto one person. Old Town
  and Uptown *exported* — Vesper 72 → 10 pings while outsiders took 52;
  Nightwell's out-of-area went 3/wk → **12 of 21**. Invisible in acceptance
  rate: a ping taken 40 minutes away counts the same as one next door.
- **Break-even is 1.5 taken per refusal.** The four ran 2.4–5.4 before August,
  about **1.0 now** — so a floor-only fix (prototype "Fix B") may never recover
  Meteor Mite or Farlight, and delays relief for The Gale and Nightwell too.
- **Acceptance rate, measured:** 76.6% → 64.0%; misses 2.3% → 18.0%. Restoring
  90s puts it back to ~76% **while the four stay at zero** — the prototype
  shows both numbers side by side deliberately.
- Agreed order (timer last, on purpose): miss ≠ refusal → un-stick the four →
  record delivery/time-to-accept → check Supply's gear scheduling → tell the
  handler → then decide the timer.
- **Still open:** ease-back rate unchosen; Security Policy 4.1 unread; Sofia
  hasn't seen the design; no channel to responders at all; not on the Q3
  roadmap; Availability Confidence conversation with Helen still hasn't
  happened; and the availability finding (most callouts have one candidate)
  needs following.
- **Caveat carried forward:** the ping data stops **6 September** and it is now
  November. Everything above is two months stale — ask Ravi Menon for current
  numbers before acting.

**Session 6 findings (Module 6).** Built `review-checklist` as a project skill
(`.claude/skills/review-checklist/SKILL.md`) and put it on a Monday 09:06
schedule. Run reports go to `06-sidekicks/review-checklist-runs/`.

- **Graded all four of Priya's wiki briefs. Nine flags, not four.** Every one
  of them fails on evidence: **not a single figure between them.** The case for
  each rests on one anecdote — "Halloran's asked more than once", "Aunt Dot
  asked for", "the support lead's team has hit this more than once".
- **Requisition Approval Chains** (14 Jul) is the worst. Says "That's the whole
  ask", then adds four things in the next sentence: 48-hour route-around, a
  cross-site pending dashboard, handler-visible status, and replacing
  Halloran's gear-cage spreadsheet. Ships-together-or-phased is "TBD".
- **Routing Override Audit Log** (5 Mar) still reads "ready to build" and
  "once profile redesign wraps" — but 4.0 (7 Apr) shipped *both* profile
  redesign and a routing override audit log. It may have been describing
  finished work for seven months. **Check the 4.0 release notes before acting
  on it.**
- **Handler Phone App** (8 Sep) opens with the app, not the problem, and the
  build is explicitly "not yet staffed". Bulk Callout (12 Jan) has no owner at
  all — "nobody's picked it up yet".
- **`06-sidekicks/scheduled-run-output.txt` is course-supplied fiction.** Dated
  13 Oct, eleven days before this repo's first commit, for a skill and a briefs
  folder that didn't exist. It also undercounts: one flag per brief where the
  real count is two or three. Never treat it as a result.
- Module 5 brief re-graded at 1 flag of 5 — no owner named. Step 4 of its plan,
  "Check the gear", has nobody on the Supply side attached to it.
