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
