# Module 3 — Data research: what Release 4.2 did

Session date: 1 November 2026
Sources: `rook-database` connector (`pings`, `support_tickets`, `callouts`,
`responders`, `handlers`), `00-rook/code/dispatch-routing/` source files.

This file records everything established in the Module 3 session. It builds on
`overview_questions.md` (Module 1) and confirms, corrects and extends it.

---

## 1. The headline finding

**Release 4.2 shipped on 12 August 2026. On that exact day, missed pings went
from ~2% to ~18%. Refusals did not rise — they fell.**

The cause is `OFFER_TIMEOUT_SECONDS`, cut from 90 to 60 seconds. The amplifier
is a scoring rule that penalises a miss exactly like a refusal and never lets
the score recover. Together they removed four of the roster's best responders
from service, permanently, with nobody deciding to do it.

### Before and after, whole roster

| Period | Pings | Taken | Turned down | Missed |
| --- | ---: | ---: | ---: | ---: |
| Before (29 Jun – 11 Aug, 44 days) | 1,085 | 831 (76.6%) | 229 (21.1%) | 25 (**2.3%**) |
| After (12 Aug – 6 Sep, 26 days) | 611 | 391 (64.0%) | 110 (18.0%) | 110 (**18.0%**) |

Misses rose roughly 8×. Refusals fell by 3 points. Weekly ping volume barely
moved (~172/wk before, ~165/wk after), so this is not a volume artefact.

### Weekly

| Week starting | Pings | Taken | Turned down | Missed | % taken | % missed |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 29 Jun | 172 | 132 | 37 | 3 | 76.7 | 1.7 |
| 6 Jul | 170 | 131 | 35 | 4 | 77.1 | 2.4 |
| 13 Jul | 174 | 133 | 35 | 6 | 76.4 | 3.4 |
| 20 Jul | 176 | 132 | 39 | 5 | 75.0 | 2.8 |
| 27 Jul | 170 | 132 | 34 | 4 | 77.6 | 2.4 |
| 3 Aug | 172 | 134 | 36 | 2 | 77.9 | 1.2 |
| **10 Aug** (release week) | 177 | 96 | 43 | 38 | 54.2 | **21.5** |
| 17 Aug | 158 | 104 | 26 | 28 | 65.8 | 17.7 |
| 24 Aug | 162 | 108 | 30 | 24 | 66.7 | 14.8 |
| 31 Aug | 165 | 120 | 24 | 21 | 72.7 | 12.7 |

Six pre-release weeks sit in a tight band: 75.0–77.9% taken, 1.2–3.4% missed.
Then it steps. Nothing drifts.

### Daily, across the release

| Day | Pings | Missed | % missed |
| --- | ---: | ---: | ---: |
| 7 Aug | 30 | 0 | 0.0 |
| 8 Aug | 23 | 1 | 4.3 |
| 9 Aug | 16 | 0 | 0.0 |
| 10 Aug | 32 | 0 | 0.0 |
| 11 Aug | 19 | 1 | 5.3 |
| **12 Aug** | 25 | 7 | **28.0** |
| 13 Aug | 25 | 12 | 48.0 |
| 14 Aug | 30 | 8 | 26.7 |
| 15 Aug | 24 | 6 | 25.0 |
| 16 Aug | 22 | 4 | 18.2 |
| 17 Aug | 24 | 1 | 4.2 |

Zero, zero, zero, then 28% on the release day itself. Seasonality does not
arrive between a Tuesday and a Wednesday. This is the single chart that ends
the "August is always soft" reading.

---

## 2. Turned down vs missed — the distinction that matters

The `pings.outcome` column holds three values. The column description says it
plainly: `taken` (they said yes), `turned down` (they said no), `missed` (no
answer in time).

| | turned down | missed |
| --- | --- | --- |
| What the responder did | Tapped "no" | Nothing |
| Who ended the ping | The responder | The clock |
| What it tells you | They saw it and declined | You don't know if they ever saw it |
| Effect on the callout | Goes to next responder | Goes to next responder |
| Recorded | Separately | Separately |

**The gap between consecutive pings on the same callout is the fingerprint:**

- Callout 40983 — turned down 14:02:33, next ping 14:02:48. **15 seconds.**
- Callout 40982 — missed 11:41:57, next ping 11:42:58. **61 seconds.**

Short gap = a human decided. Long, identical gap = a countdown expired. Before
4.2 the miss gap was ~92s; after, ~62s.

**`missed` does not mean "ignored".** It means Dispatch didn't hear back in
time — asleep, driving, no signal, phone in a bag, notification never
delivered, or saw it and hesitated two seconds too long. All recorded
identically.

A refusal is evidence about a person. A miss is evidence about a
delivery-and-timing system, and only *maybe* about a person. The code treats
them as the same thing.

---

## 3. Timeout vs delivery — how far the data can take us

An open question was whether the misses were caused by the shortened timeout
or by a push-delivery regression also shipped in 4.2.

### What the pings table cannot do

`pings` has no delivery field and no time-to-accept. When Dispatch writes
`missed`, that covers two physically different events:

- the push arrived, buzzed, and the 60-second timer expired
- the push never arrived, and the timer expired because nothing could arrive

**These produce byte-identical rows.** No query can separate them.

### Discriminating tests run

**By time of day** — the rise is uniform, not concentrated at night:

| Time block | % missed before | % missed after |
| --- | ---: | ---: |
| 00–04 | 6.3 | 13.0 |
| 04–08 | 0.0 | 16.2 |
| 08–12 | 2.1 | 18.6 |
| 12–16 | 1.9 | 14.3 |
| 16–20 | 2.6 | 21.4 |
| 20–24 | 1.8 | 17.6 |

**By responder** — all 16 rose. This kills the narrow version (a carrier or
device fault would hit a subset) but not the broad version (a server-side push
regression would hit everyone).

**Burst test** — stripping out the four suppressed responders, the other twelve
scatter day-to-day within roughly binomial noise. **No outage signature.**

**The natural experiment:** Stormwrack (0 misses in 82 pings), Sgt. Bulwark
(0 in 57) and Halfmoon (0 in 69) had **zero misses in 208 pings across 44
days**. After 12 August: **21 misses in 135 pings — 15.6%**. Something
external changed. This rules out responder behaviour; it does not say which
external thing.

### What settled it: the tickets

Every push/notification/signal complaint in the dataset **predates the
release**:

| Date | Subject | About |
| --- | --- | --- |
| 6 Jul | Repeated notification | The Longcast |
| 14 Jul | Missed one while out of signal | The Undertow |
| 4 Aug | Push notifications unreliable on older phones | Ironvale |
| 10 Aug | Double notifications | Ironvale |

**Zero after 12 August.** Handlers demonstrably know how to file a delivery
complaint — they did it four times before. After the release they stop, and
their silence complaints are framed as *duration* ("not gone off in 7 days",
"Is Farlight still on the list?"), which is a roster/ranking suspicion, not a
broken phone.

**Conclusion: the timeout is the cause and must carry the full magnitude.**
Delivery is not formally excluded — absence of evidence from a channel with a
known blind spot — but it is now the weaker hypothesis by a distance.

### The honest residual doubt

For a 30-second trim to take misses from 2.3% to 18%, roughly 16% of all pings
must be answered in the 60–90 second band while only 2.3% take longer than 90
seconds. That is a lot of weight in a narrow window. It is defensible — "hear
the buzz, finish the sentence, find the phone, unlock, tap" plausibly clusters
near a minute — but it is an assumption, and only time-to-accept data can
confirm it.

---

## 4. Per-responder breakdown

Before = 44 days, after = 26 days. A volume ratio of **0.59** means unchanged.

| Responder | Handler | Area | Pings before → after | Volume change | % missed before → after |
| --- | --- | --- | ---: | ---: | ---: |
| Farlight | Linda Pruitt | Uptown | 75 → 11 | **−75%** | 4.0 → 63.6 |
| The Undertow | Desmond Okafor | Harborside | 75 → 14 | **−68%** | 1.3 → 64.3 |
| Vesper | **Aunt Dot** | Old Town | 86 → 17 | **−67%** | 3.5 → 52.9 |
| Meteor Mite | **Kip** | Eastgate | 70 → 14 | **−66%** | 2.9 → 57.1 |
| Corporal Ashgrove | Yusuf Demir | Northfield | 63 → 29 | −22% | 1.6 → 13.8 |
| Halfmoon | Simone Fischer | Westbury | 69 → 33 | −19% | 0.0 → 15.2 |
| Sgt. Bulwark | Halloran | Foundry Row | 57 → 39 | +16% | 0.0 → 17.9 |
| The Longcast | Graham Petrov | Lakeshore | 45 → 32 | +20% | 2.2 → 15.6 |
| The Drift | Beatrice Calloway | Greenway | 39 → 28 | +21% | 2.6 → 17.9 |
| Cindermark | Farid Haddad | Mill District | 57 → 42 | +25% | 1.8 → 11.9 |
| Nightwell | Marjorie Sung | Midtown | 95 → 70 | +25% | 3.2 → 11.4 |
| Ironvale | Teresa Alvarez | Riverside | 51 → 39 | +29% | 2.0 → 17.9 |
| Stormwrack | Renata Kovač | Southport | 82 → 63 | +30% | 0.0 → 14.3 |
| Captain Vantage | Mr. Ambrose | Hillcrest | 76 → 59 | +31% | 2.6 → 10.2 |
| Sgt. Falkirk | Owen Bramwell | Kingsbridge | 63 → 52 | +40% | 3.2 → 15.4 |
| The Gale | **Kip** | Eastgate | 82 → 69 | +42% | 4.9 → 11.6 |

Group A's share of all pings fell from **28.2% to 9.2%**. Those 19 points went
almost entirely to the top of the ranking.

Caveat: the 53–64% miss rates in group A sit on only 11–17 pings each. The
direction is unambiguous; don't quote the percentages as precise.

**Turn-down rate did not rise for a single responder.** Before range
12.8–28.6%; after 14.3–21.4%. Every point of lost acceptance in every group
went into `missed`.

---

## 5. Tickets vs pings — do they agree?

`support_tickets`: 147 rows, 29 June – 7 September, with `filed_by` (handler),
`about_responder`, `subject`, `body`, `status`.

### They agree on the date, disagree on the peak

| Week | % missed | Tickets |
| --- | ---: | ---: |
| 29 Jun – 3 Aug (6 weeks) | 1.2 – 3.4 | 5–8/wk |
| **10 Aug** | **21.5** ← peak | 20 |
| 17 Aug | 17.7 | 27 |
| **24 Aug** | 14.8 | **32** ← peak |
| 31 Aug | 12.7 | 25 |

Baseline 6.5/wk → ~26/wk, i.e. **4×** (slightly above Nadia's "~3×"). **The
two series move in opposite directions after 17 August.** Misses fall every
week from the 10 August peak; tickets keep climbing for two more. Watch
acceptance and you report recovery; watch tickets and you report
deterioration. Both are correct and both are incomplete.

### The two themes are sequential, not parallel — the key structural finding

| Week | A: "gone before they could answer" | B: "phone never goes off" | Other | Total |
| --- | ---: | ---: | ---: | ---: |
| 29 Jun – 3 Aug | **0** | **0** | 5–8/wk | 39 |
| 10 Aug | **7** | 0 | 13 | 20 |
| 17 Aug | 3 | **8** | 16 | 27 |
| 24 Aug | 3 | **12** | 17 | 32 |
| 31 Aug | 2 | 8 | 15 | 25 |
| 7 Sep | 0 | 2 | 2 | 4 |

- **Theme A first appears 12 August** — *"Ping moved on while The Drift was
  reaching for the phone"*. Day zero. 15 tickets, peaks immediately, decays.
- **Theme B first appears 17 August** — *"Farlight's phone has not gone off in
  3 days"*. **Five days later.** 30 tickets, rising for two weeks.

Nadia described two simultaneous buckets. They are **consecutive**, and the
sequence is the causal chain: timeout → misses immediately → penalties
accumulate → suppression ~5 days later. The 15/30 split also reproduces her
one-third / two-thirds ratio, so her volumes were right.

**This is the strongest confirmation available, because tickets know nothing
about `config.py`.**

### Where they disagree hardest: who

| Ticket leaderboard | | Damage leaderboard | |
| --- | ---: | --- | ---: |
| The Undertow | 21 | Farlight | −75% |
| Farlight | 19 | The Undertow | −68% |
| Halfmoon | 15 | **Vesper** | **−67%** |
| Corporal Ashgrove | 14 | **Meteor Mite** | **−66%** |

**Vesper and Meteor Mite filed zero tickets. Ever.** Their handlers — Aunt Dot
and Kip — have never used the ticket system. Kip handles *both* the worst-
affected responder (Meteor Mite) and the best (The Gale) and files nothing
either way, so Eastgate is invisible in both directions.

Linda Pruitt and Desmond Okafor filed 16–17 tickets each for comparable harm.
Aunt Dot and Kip would have filed roughly 30 more. **About a fifth of the true
signal never reached the ticket system at all.**

Triaging by ticket volume in late August would have meant investigating the
two mildest cases and never looking at the two silent catastrophes.

### Caveat on the headline ticket number

Part of the surge is filter noise, exactly as Priya predicted. Filter tickets:
0–1/wk before, then 3, **7**, 2, 2 after. The routing-specific themes went
from **0/wk to 7–15/wk** — that is the number that matters.

### Subject lines worth quoting

- **"First one in 9 days and he missed it"** (5 Sep) — the ratchet in one
  sentence: suppressed nine days, then penalised again for the one ping he got.
- "First one in 5 days and she missed it" (29 Aug)
- "Is The Undertow still on the list?" (30 Aug) / "Is Farlight still on the
  list?" (28 Aug) — they are on the list, ranked to the bottom of it, which is
  indistinguishable from removal and invisible in the console.
- "The Undertow asked me to pass this on" / "Farlight asked me to pass this
  on" (both 1 Sep) — responders have no channel; they must route through a
  handler, and only if that handler files.

---

## 6. The three groups

Groups are defined by what happened *after* the release, not by anything true
before it.

- **A collapsed** (4): Vesper, Farlight, The Undertow, Meteor Mite
- **B drifting** (2): Corporal Ashgrove, Halfmoon
- **C absorbing** (10): everyone else

| Group | Period | People | Pings | Taken | Turned down | Missed | Per person/day |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| A | before | 4 | 306 | 231 (75.5%) | 66 (21.6%) | 9 (2.9%) | 1.74 |
| A | after | 4 | 56 | 13 (23.2%) | 10 (17.9%) | 33 (**58.9%**) | **0.54** |
| B | before | 2 | 132 | 99 (75.0%) | 32 (24.2%) | 1 (0.8%) | 1.50 |
| B | after | 2 | 62 | 40 (64.5%) | 13 (21.0%) | 9 (14.5%) | 1.19 |
| C | before | 10 | 647 | 501 (77.4%) | 131 (20.2%) | 15 (2.3%) | 1.47 |
| C | after | 10 | 493 | 338 (68.6%) | 87 (17.6%) | 68 (13.8%) | **1.90** |

### Workload trajectory — pings per person per day, by week

| Week | A | B | C |
| --- | ---: | ---: | ---: |
| 29 Jun | 1.75 | 1.50 | 1.46 |
| 6 Jul | 1.71 | 1.50 | 1.44 |
| 13 Jul | 1.71 | 1.50 | 1.50 |
| 20 Jul | 1.79 | 1.50 | 1.50 |
| 27 Jul | 1.79 | 1.50 | 1.41 |
| 3 Aug | 1.75 | 1.50 | 1.46 |
| 10 Aug | 1.54 | 1.50 | 1.61 |
| 17 Aug | 0.57 | 1.21 | 1.79 |
| 24 Aug | 0.21 | 1.07 | 2.01 |
| 31 Aug | **0.11** | 1.07 | **2.10** |

### Miss rate by week

| Week | A | B | C |
| --- | ---: | ---: | ---: |
| 29 Jun – 3 Aug | 0.0 – 6.3 | 0.0 – 4.8 | 1.0 – 3.0 |
| 10 Aug | 39.5 | 14.3 | 15.9 |
| 17 Aug | 56.3 | 17.6 | 12.8 |
| 24 Aug | 66.7 | 13.3 | 12.8 |
| 31 Aug | 100.0 | 6.7 | 11.6 |

### The most important point in the whole analysis

**Before 12 August the three groups were the same group.** Workload 1.74 /
1.50 / 1.47. Taken 75.5% / 75.0% / 77.4%. Missed 2.9% / 0.8% / 2.3%. Three
flat, overlapping lines for six weeks.

There was no weak tier waiting to wash out. **Group A was carrying the
heaviest load of the three before the release.** The system did not find weak
responders — it manufactured them, out of its four busiest people, in under a
fortnight.

Caveats: group A's 100% in the final week sits on ~3 pings — read it as
"effectively out of rotation", not as a rate.

**Correction — Group B is not being suppressed.** See section 6a. Once
normalised against incident volume in their own areas, Corporal Ashgrove and
Halfmoon are **up** 14% and 16%. Their raw decline is a demand effect, not the
ratchet. The "drifting" label describes their raw ping count, not their
standing.

---

## 6a. Proximity and geography — tested, and ruled out

`WEIGHT_PROXIMITY` went 0.45 → 0.60 in 4.2, so a fair hypothesis is that the
suppressed responders simply sit where the incidents stopped happening.
**They don't.**

### What location data exists

There are **no coordinates anywhere**. `callouts` has `area` (text) and
`what_happened`; `responders` has `area` and `handler`. So a geographic map
cannot be drawn without inventing positions, and none was. Area-level analysis
is all the data supports.

### Incidents per day by area

Every area declined modestly — the general July-to-August dip — but the
declines do not line up with who was suppressed:

| Area | Responder | Incidents/day before → after | Change |
| --- | --- | ---: | ---: |
| Northfield | Corporal Ashgrove | 1.18 → 0.81 | **−32%** |
| Westbury | Halfmoon | 1.27 → 0.88 | **−30%** |
| Greenway | The Drift | 0.82 → 0.62 | −25% |
| Mill District | Cindermark | 1.05 → 0.85 | −19% |
| Eastgate | Meteor Mite **and** The Gale | 2.68 → 2.19 | −18% |
| Foundry Row | Sgt. Bulwark | 1.00 → 0.85 | −15% |
| Midtown | Nightwell | 1.73 → 1.46 | −15% |
| Harborside | The Undertow | 1.43 → 1.23 | −14% |
| Hillcrest | Captain Vantage | 1.34 → 1.15 | −14% |
| Kingsbridge | Sgt. Falkirk | 1.23 → 1.08 | −12% |
| Old Town | Vesper | 1.64 → 1.46 | −11% |
| Uptown | Farlight | 1.34 → 1.19 | −11% |
| Lakeshore | The Longcast | 0.80 → 0.73 | −8% |
| Riverside | Ironvale | 0.93 → 0.88 | −5% |
| Southport | Stormwrack | 1.55 → 1.54 | 0% |

The four collapsed responders are in areas that lost **11–18%** of incidents.
The two "drifting" ones are in the areas that lost the **most** (−32%, −30%).

### Demand-adjusted: pings per local incident

Dividing each responder's pings per day by incidents per day in their own area
removes the demand effect entirely.

| Responder | Area | Raw ping change | Demand-adjusted |
| --- | --- | ---: | ---: |
| Farlight | Uptown | −75% | **−72%** |
| Vesper | Old Town | −67% | **−63%** |
| The Undertow | Harborside | −68% | **−63%** |
| Meteor Mite | Eastgate | −66% | **−59%** |
| Corporal Ashgrove | Northfield | −22% | **+14%** |
| Halfmoon | Westbury | −19% | **+16%** |
| Stormwrack | Southport | +30% | +31% |
| The Longcast | Lakeshore | +20% | +31% |
| Ironvale | Riverside | +29% | +36% |
| Sgt. Bulwark | Foundry Row | +16% | +37% |
| Nightwell | Midtown | +25% | +47% |
| Captain Vantage | Hillcrest | +31% | +53% |
| Cindermark | Mill District | +25% | +54% |
| Sgt. Falkirk | Kingsbridge | +40% | +59% |
| The Drift | Greenway | +21% | +62% |
| The Gale | Eastgate | +42% | **+74%** |

**Two conclusions:**

1. **The collapse survives the adjustment almost untouched** (−59% to −72%).
   Demand explains a few points of it and nothing more.
2. **Group B is not suppressed at all.** Ashgrove and Halfmoon are *up* 14%
   and 16% per local incident. Their raw decline is entirely explained by their
   areas having the two largest incident drops. This corrects the earlier
   reading in section 6 that they might be "early on Group A's curve" — they
   are not on that curve.

### The decisive control: Eastgate

**Meteor Mite and The Gale cover the same area, with the same handler (Kip),
drawing on the same incidents.**

- Meteor Mite: **−59%** demand-adjusted
- The Gale: **+74%** demand-adjusted

Same geography, opposite outcomes, 133 points apart. No proximity-based
explanation can survive this. (It confirms the Module 1 finding, now measured.)

### A side effect worth noting: cascades got longer

| | Before | After |
| --- | ---: | ---: |
| Pings outside the responder's own area | 186 (17.1%) | 241 (**39.4%**) |
| Pings per callout | 1.23 | **1.39** |

This looks backwards for a release that *increased* the proximity weight — and
it is the ratchet showing up in the geography. When the local responder misses,
the callout cascades down the ranked list to someone further away. More misses
means longer cascades means more out-of-area pinging. **Raising the proximity
weight has produced less local coverage, not more**, because the mechanism that
decides who is asked first keeps removing the nearest person.

That is worth raising with Wen Li alongside everything else: the weight change
is not delivering what it was requested for, and the reason is concern 4, not
the weight itself.

---

## 6b. Marcus's question answered — refusals never mattered, misses did

Marcus asked on 14 August whether the change to who gets pinged was meant to
apply to responders who have been turning jobs down, or only to everyone else,
since the config doesn't distinguish. This section answers it with evidence.

**The claim tested:** the change applied to everyone's existing score, but
before 4.2 refusal history had been wiped out by the 1.0 ceiling, so old
history made almost no difference; after 4.2, refusing didn't sink anyone
either; what decided it was missed pings, because `offer.py:30` counts a miss
as a turn-down. So the change did not hit people who turn jobs down. It hit
people who didn't answer within 60 seconds.

**All three clauses hold.**

### Clause 1 — the ceiling erased refusal history

`SCORE_CEILING = 1.0` and `_set()` clamps. Every taken ping while already at
1.00 earns nothing. Across the roster, **315 acceptance credits were thrown
away** this way before 12 August, and **all 16 responders reached the ceiling**.

| Responder | Refusals before 4.2 | Score on 11 Aug | Credits discarded |
| --- | ---: | ---: | ---: |
| Meteor Mite | **20** | **1.00** | 5 |
| Nightwell | 20 | 0.96 | 26 |
| Halfmoon | 19 | 0.88 | 14 |
| Captain Vantage | 18 | 0.96 | 19 |
| Farlight | 17 | 0.84 | 18 |
| Stormwrack | 17 | 0.88 | 32 |
| The Undertow | 16 | 0.88 | 25 |
| Sgt. Bulwark | 14 | 0.80 | 16 |
| The Gale | 14 | 0.96 | 29 |
| Vesper | 13 | 0.96 | 35 |
| Corporal Ashgrove | 13 | 0.88 | 21 |
| Sgt. Falkirk | 11 | **1.00** | 19 |
| Cindermark | 11 | 0.96 | 18 |
| Ironvale | 11 | **1.00** | 13 |
| The Longcast | 10 | **1.00** | 9 |
| The Drift | **5** | **1.00** | 16 |

**No relationship between refusals and score.** Refusals range 5–20; scores
range 0.80–1.00; the correlation is noise. Meteor Mite refused four times as
many callouts as The Drift and they were both at 1.00.

Old refusal history was not weighted lightly — it was **deleted**.

### Clause 2 — post-4.2 refusals didn't sink anyone either

Nobody's refusal rate rose after the release (see section 4: before
12.8–28.6%, after 14.3–21.4%). The heaviest post-release refusers all did
well.

### Clause 3 — misses decided it

### The three examples

| | Refused most | Kept refusing after | Refused least |
| --- | --- | --- | --- |
| | **Meteor Mite** 28.6% | **Captain Vantage** 18.6% | **Vesper** 15.1% |
| Refusals before | 20 (highest on roster) | 18 | 13 (2nd lowest) |
| Score on 11 Aug | **1.00** | 0.96 | 0.96 |
| Misses after 4.2 | 8 (57.1%) | 6 (10.2%) | 9 (52.9%) |
| Ping volume change | **−66%** | **+31%** | **−67%** |
| Final score | **0.00** | 0.92 | **0.00** |

The heaviest refuser and the lightest refuser ended in the same place: the
floor. The responder who kept refusing after the release gained a third more
work. **Refusals are not the variable.**

### The break-even: 60%

`ACCEPTANCE_CREDIT = 0.08`, `DECLINE_PENALTY = 0.12`. Setting them equal:

```
0.08 × T = 0.12 × (1 − T)   →   0.20 T = 0.12   →   T = 60%
```

**Above a 60% taken rate you climb to the ceiling and pin there. Below it you
fall to the floor. There is no stable middle.**

Taken rate after 12 August, sorted:

| Below break-even | | Above break-even | |
| --- | ---: | --- | ---: |
| Farlight | 18.2% | Halfmoon | 63.6% |
| Meteor Mite | 21.4% | The Drift | 64.3% |
| The Undertow | 21.4% | Corporal Ashgrove | 65.5% |
| Vesper | 29.4% | The Longcast | 65.6% |
| | | Ironvale, Sgt. Bulwark | 66.7% |
| | | Nightwell | 67.1% |
| | | Cindermark | 69.0% |
| | | Sgt. Falkirk | 69.2% |
| | | The Gale | 69.6% |
| | | Captain Vantage | 71.2% |
| | | Stormwrack | 71.4% |

**Nobody sits between 29.4% and 63.6%.** The distribution is bimodal with an
empty gap, exactly as a 60% break-even with no decay predicts.

**The decisive arithmetic:** to cross the break-even on refusals alone you
would have to refuse **40%** of your pings. The highest refusal rate ever
recorded here is Meteor Mite's 28.6%, which leaves 71.4% taken — comfortably
safe. **Refusals cannot sink anyone under these constants.** Misses can,
because they stack on top: Vesper's 17.6% refusals *plus* 52.9% misses leaves
29.4% taken.

### The counterfactual

Replaying every responder with misses carrying no penalty (refusals still
−0.12):

| Responder | Actual final | If misses were free |
| --- | ---: | ---: |
| Vesper | 0.00 | **0.76** |
| The Undertow | 0.00 | **0.84** |
| Farlight | 0.00 | **0.80** |
| Meteor Mite | 0.00 | **0.84** |

**Nobody reaches the floor — 0 of 16.** The four would be mid-pack and still
in rotation.

Caveat: the counterfactual replays their *actual* ping sequence, which is
already starved because the real system stopped pinging them. With a healthy
score they would have kept receiving pings and kept earning credit, so
0.76–0.84 is the floor of the counterfactual, not the ceiling. It understates
the case.

### The answer to give Marcus

> It applied to everyone's score, and it did not hit the people who turn jobs
> down — it couldn't. Refusal history was already erased by the ceiling, and
> no refusal rate we have ever recorded is high enough to cross the 60%
> break-even. What crossed it was misses, because `offer.py:30` counts a miss
> as a turn-down.

So the config not distinguishing between a miss and a refusal is **the bug**,
not an oversight in how the change was applied. That reframes his question:
it was never a question about the weighting, it was a question about concern 3.

---

## 7. Vesper — the case study

Handler: **Aunt Dot**. Area: Old Town. **Zero support tickets, ever.**

| Week starting | Pings | Taken | Turned down | Missed | Score at week end | Rank of 16 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 29 Jun | 14 | 11 | 3 | 0 | 0.88 | **2nd** |
| 6 Jul | 13 | 10 | 2 | 1 | **1.00** (ceiling) | **2nd** |
| 13 Jul | 15 | 13 | 1 | 1 | **1.00** | **1st** |
| 20 Jul | 14 | 11 | 3 | 0 | **1.00** | **2nd** |
| 27 Jul | 13 | 11 | 1 | 1 | 0.88 | **2nd** |
| 3 Aug | 14 | 12 | 2 | 0 | 0.96 | **2nd** |
| 10 Aug | 12 | 6 | 2 | **4** | 0.68 | 5th |
| 17 Aug | 5 | 1 | 1 | 3 | 0.28 | 13th |
| 24 Aug | 2 | 0 | 1 | 1 | 0.04 | 13th |
| 31 Aug | 1 | 0 | 0 | 1 | **0.00** (floor) | 13th |

Rank = position by pings received that week — the observable consequence of
the score.

### Every ping from 10 August

| Callout | When | Outcome |
| --- | --- | --- |
| 40938 | Mon 10 Aug, 02:01 | taken |
| 40945 | Mon 10 Aug, 13:33 | turned down |
| 40975 | Tue 11 Aug, 21:00 | taken |
| — | **Release 4.2, Wed 12 August** | — |
| 40992 | Thu 13 Aug, 07:34 | **missed** |
| 41006 | Fri 14 Aug, 02:53 | taken |
| 41017 | Fri 14 Aug, 17:10 | **missed** |
| 41023 | Sat 15 Aug, 13:02 | taken |
| 41026 | Sat 15 Aug, 15:33 | turned down |
| 41034 | Sat 15 Aug, 20:58 | taken |
| 41039 | Sun 16 Aug, 10:46 | **missed** |
| 41042 | Sun 16 Aug, 14:05 | **missed** |
| 41043 | Sun 16 Aug, 16:03 | taken |
| 41050 | Mon 17 Aug, 08:55 | taken |
| 41102 | Wed 19 Aug, 18:42 | **missed** |
| 41153 | Sun 23 Aug, 00:10 | **missed** |
| 41155 | Sun 23 Aug, 05:34 | turned down |
| 41165 | Sun 23 Aug, 20:15 | **missed** |
| 41186 | Mon 24 Aug, 21:40 | turned down |
| 41214 | Wed 26 Aug, 18:09 | **missed** |
| 41311 | Tue 1 Sep, 11:42 | **missed** |

**Four misses between 13 and 16 August** — the threshold. But in those same
five days Vesper also **took five callouts**. Vesper was answering. Four times
out of thirteen the sixty seconds ran out, and that was enough.

The four fatal misses fell on four different days (hours 79.6, 113.2, 154.8,
158.1 of that week — Thursday, Friday, Saturday, Saturday night). Not one bad
evening. That is consistent with a window too short for ordinary life, and
inconsistent with anything about Vesper having changed.

### The silence, measured in callouts

- 17 Aug → 19 Aug: **52 callouts** went past
- 19 Aug → 23 Aug: **51 callouts**
- 26 Aug → 1 Sep: **96 callouts**
- Since 1 Sep: **108 callouts**, and counting

Last ping: **1 September**, a miss. **Zero pings in the final five days** of
the data.

Vesper took **70** callouts before the release — second only to Nightwell's 72
— at 81.4% acceptance. Vesper now takes **5**.

---

## 8. Four responders compared

Two who went quiet, two who got busier. Pings per week, and the replayed
score at the end of each week.

| Responder | Handler | Area | Group |
| --- | --- | --- | --- |
| Vesper | Aunt Dot | Old Town | went quiet |
| Farlight | Linda Pruitt | Uptown | went quiet |
| The Gale | Kip | Eastgate | got busier |
| Sgt. Falkirk | Owen Bramwell | Kingsbridge | got busier |

### Replayed score, end of week

| Week | Vesper | Farlight | The Gale | Sgt. Falkirk |
| --- | ---: | ---: | ---: | ---: |
| 29 Jun | 0.88 | 0.86 | 0.84 | 1.00 |
| 6 Jul | 1.00 | 1.00 | 1.00 | 1.00 |
| 13 Jul | 1.00 | 1.00 | 0.96 | 1.00 |
| 20 Jul | 1.00 | 1.00 | 1.00 | 1.00 |
| 27 Jul | 0.88 | 1.00 | 0.84 | 1.00 |
| 3 Aug | 0.96 | 0.80 | 1.00 | 1.00 |
| 10 Aug | 0.68 | 0.40 | 0.84 | 1.00 |
| 17 Aug | 0.28 | 0.04 | 1.00 | 0.88 |
| 24 Aug | 0.04 | **0.00** | 1.00 | 1.00 |
| 31 Aug | **0.00** | **0.00** (no pings) | 0.96 | 0.76 |

### Misses in the release week — the whole difference

| Responder | Misses, week of 10 Aug | Outcome |
| --- | ---: | --- |
| Vesper | 4 | removed |
| Farlight | 4 | removed |
| The Gale | 3 | recovered to 1.00 |
| Sgt. Falkirk | 2 | never dropped |

**Two misses, in one week, separate the people who survived from the people
who were removed.** The Gale and Sgt. Falkirk recovered because they kept
getting pinged — pings are the only source of the +0.08 credit. Vesper and
Farlight fell just far enough that the ranking cut their supply, and once the
supply is cut the score cannot rise.

### Two further observations

- **Sgt. Falkirk ends on 0.76** — his first drop in the whole period. He is at
  the top of the roster by workload and is now losing score for it. More pings
  means more chances to miss one. Group C's protection is slower, not
  permanent.
- **Farlight's final week is blank** — not zero misses, zero pings. The 0.00
  is carried forward because there was no event to change it, and there never
  will be.

---

## 9. The end-to-end flow and the six concerns

### The routing loop (`00-rook/code/dispatch-routing/`)

```
Callout raised (incident in console)
        ↓
Rank available responders        routing.rank_for_callout()
  score = 0.60 × proximity
        + 0.25 × recent_acceptance
        + 0.15 × capability_match
        ↓
Ping the top responder           offer.offer_to(), 60-second window
        ↓
Record the outcome               taken / turned down / missed
        ↓
Update the score                 history.record_accepted / record_declined
        ↓
   ↻ feeds back into ranking
```

If the loop reaches the bottom of the list with no yes, `dispatch` returns
`None` — nobody came.

### Where callouts, pings and filters fit

- **A callout** is the unit of work — one incident, one callout. 1,319 in the
  data.
- **A ping** is one offer of one callout to one phone. 1,696 pings for 1,319
  callouts, because `offer.dispatch` loops down the ranked list. This is why
  acceptance is measured per ping, not per callout.
- **Filters** are a console feature. They change what a handler *sees*. They
  do not enter `rank_for_callout`, do not reach `offer_to`, and cannot affect
  who gets pinged. 4.2 made them persist; they generated 3, 7, 2, 2 tickets in
  the four weeks after release. **Ticket noise, not signal.**

### The six concerns

**1 — The 60-second window is too short.** `config.py:9`,
`OFFER_TIMEOUT_SECONDS = 60`, was 90. Proven live: 62s between consecutive
pings on a miss, vs 92s before. One line. Revertible this week.

**2 — No delivery record, no time-to-accept.** `push_to_device` fires and
nothing comes back. "Never arrived" and "didn't answer" are written down
identically. This is why the delivery question couldn't be fully closed from
data, and why you cannot pick a *correct* timer value — nobody knows the real
response-time distribution.

**3 — A miss is penalised exactly like a refusal.** `offer.py:30` calls
`history.record_declined` for both. The docstring says it out loud: *"Same
either way — we asked and we didn't get a yes."* −0.12 whether someone
declined deliberately or was in a tunnel.

With `ACCEPTANCE_CREDIT = 0.08` against `DECLINE_PENALTY = 0.12`, the
**break-even taken rate is exactly 60%** — above it you pin to the ceiling,
below it you fall to the floor, and there is no stable middle. Refusals alone
have never come close to crossing it (you would need to refuse 40% of pings;
the record is 28.6%). Misses crossed it for four people. See section 6b.

**4 — The loop closes and never reopens.** `history.py:30` carries a
`TODO(2019)` debating whether the score should ease back toward neutral. It
never got resolved, so it doesn't. Score feeds ranking → ranking decides pings
→ pings are the only way to earn score back.

The arithmetic: a miss costs 0.12, an acceptance earns 0.08. **Three yeses to
undo two misses** — paid in a currency the system has stopped issuing.

**5 — Responders cannot tell us anything.** All 147 tickets were filed by
handlers, and filing is optional. Aunt Dot and Kip have never filed one, and
between them they look after Vesper and Meteor Mite.

**6 — Supply is downstream and unverified.** `availability.py` opens with a
warning that the record Dispatch writes is read by Supply to schedule gear
servicing *"for windows when a responder isn't likely to be called out."* Four
responders have had six artificially quiet weeks. Not checkable from the
Dispatch database. Open question, not a known defect — but it has a safety
edge.

### A note on the weight change

`WEIGHT_RECENT_ACCEPTANCE` went 0.40 → 0.25 in 4.2. This is commonly read as
"acceptance matters less now, so it's less dangerous." The opposite is true
for a high scorer: at 0.40, a ceiling score of 1.00 contributed 0.40 to the
ranking — a large cushion. At 0.25 the cushion shrank to 0.25 *before*
anything went wrong. Less protection on the way down, for exactly the
responders who had earned the most of it.

---

## 10. What to do, and in what order

### The framing

**The timer is a dial. The punishment scheme is a trap.** Dials can be turned
back and forth. Traps must be removed before you dare touch anything, because
every experiment risks another permanent casualty. Fixing the trap converts an
irreversible decision into a reversible one.

### Two corrections to the obvious plan

1. **The score reset must come *after* the punishment fix, not before.** Reset
   Vesper today with the timer at 60 and penalties unchanged, and Vesper is
   back on the floor inside a week — having been told it was fixed.
2. **"Defer the timer decision" must not mean "leave 60 seconds running."**
   Separate the *tourniquet* (back to 90: one line, reversible, known-good)
   from the *decision* (what the value should be). The ratchet is still
   turning; Halfmoon and Ashgrove are sliding now.

### On "penalise no, but not miss"

Don't take the miss penalty to zero — a responder with a dead phone would sit
at rank one indefinitely, burning the full timeout on every callout and
pushing "nobody came" higher still.

Better framing: **a refusal and a miss are different kinds of fact with
different half-lives.** A refusal is information about *willingness* and may
reasonably persist. A miss is information about *reachability right now* and
expires in hours. So a miss should be a short, automatically-expiring cooldown
— not a demerit. That makes this whole class of bug impossible rather than
merely smaller, and it doesn't need the delivery field to implement.

### Sequence

**Now, in parallel, this week:**

1. **Wen Li** — was miss-equals-refusal intended, and does any tool exist to
   reset a score? Her answer determines whether step 7 is even possible. If
   there is no reset tool, the fix is much bigger than budgeted and Helen
   needs to know immediately. This also finally answers Marcus's 14 August
   question.
2. **Timer back to 90** as a tourniquet. Say plainly it is temporary and not
   the decision.
3. **Tell the handlers** — all six affected, plus Aunt Dot and Kip unprompted.
4. **Supply** — is the gear of the four quiet responders still being serviced?
   Independent; don't serialise it.

**Then:**

5. **Fix the scoring** — miss becomes an expiring cooldown, not a permanent
   demerit. Add decay regardless, since the `TODO(2019)` means even refusals
   never heal.
6. **Add the two fields** — delivery receipt and time-to-accept.
7. **Reset the six** — the four floored and the two sliding, once there is a
   hole-free system to reset them into.

**Then, and only then:**

8. **Decide the timer** on the response-time distribution you will finally
   have.

### Political note

Priya's refusal to reopen 4.2 was about the **weighting change**, which was
long-requested by responders covering wide areas. The **timeout cut** is a
separate component of the same release. You can revert the timer without
touching what she was defending — say so explicitly in the room, because
someone will conflate them in the first thirty seconds.

---

## 11. Caveats — read before quoting anything

- **There is no score column in the database.** The score lives in memory in
  `history.py` (`_scores = {}`). Every score in this file is a
  **reconstruction**, replaying that responder's ping outcomes through the
  real constants: start at `NEUTRAL_SCORE = 0.5`, `+0.08` taken, `−0.12` for
  either kind of no, clamped to `[0.0, 1.0]`. Two assumptions: the 29 June
  starting value is unknown and assumed to be 0.5, and penalties are applied
  in timestamp order (which matters because of the clamp). The *levels* may be
  offset; the *shapes* and the floor at zero are not in doubt.
- **Group A's post-release percentages sit on 11–17 pings each.** Direction
  solid, precision poor.
- **Daily counts are noisy** — ~24 pings/day means one ping swings the rate by
  four points. Use the daily view for *when*, the weekly view for *how big*.
- **The database starts 29 June 2026.** No year-on-year comparison is
  possible, so Priya's seasonal hypothesis is untestable from this data. (It
  is also unnecessary: incidents did dip ~138/wk in July to 110–127 mid-August,
  but acceptance is a ratio, so a volume dip does not explain it.)
- **Rank** in this file means position by pings received that week — the
  observable consequence of the score, not the score itself.
- **The delivery hypothesis is weakened, not formally excluded.**

---

## 12. Open questions

1. **Wen Li**: was miss-equals-refusal a decision or an accident, and **does
   any tool exist to reset a responder's score?** Nothing in our materials
   shows one. (Marcus's 14 August question is now **answered analytically** —
   see section 6b: the change did not hit people who turn jobs down, it hit
   people who missed. What still needs Wen Li is whether the miss/refusal
   conflation was intended, and whether a reset tool exists. Also worth asking:
   does restoring 90s help the four at all, or are they stranded regardless?)
2. **Supply**: is gear for Vesper, Farlight, The Undertow and Meteor Mite
   still being serviced during their artificially quiet weeks? Not checkable
   from the Dispatch database. Safety-relevant.
3. **Helen**: what happened to Availability Confidence, committed to 4.2 and
   absent from the release notes? And which deferred items are still Q3
   commitments — a conversation Priya flagged as never having happened.
4. **Instrumentation**: `pings` needs a delivery receipt and a time-to-accept
   field. Without them this class of question is unanswerable every release.
5. **"Nobody came" doubled, 5.5% → 11.1%** — the only outcome in the system
   that nobody watches. It appears in no Rook document, dashboard or metric.
6. **Wen Li, second question**: out-of-area pings went 17% → 39% and pings per
   callout 1.23 → 1.39. The proximity weight was raised to improve local
   coverage and has delivered the opposite, because the ratchet keeps removing
   the nearest responder. Was this foreseen?
7. ~~Group B — early on Group A's curve?~~ **Answered.** Demand-adjusted they
   are up 14% and 16%; their raw decline is their areas' incident drop. Not
   suppressed. See section 6a.

---

## Appendix — key queries

Weekly outcome mix:

```sql
SELECT date_trunc('week', sent_at)::date AS week_start,
       COUNT(*) AS pings,
       SUM(CASE WHEN outcome='taken' THEN 1 ELSE 0 END) AS taken,
       SUM(CASE WHEN outcome='turned down' THEN 1 ELSE 0 END) AS turned_down,
       SUM(CASE WHEN outcome='missed' THEN 1 ELSE 0 END) AS missed
FROM pings GROUP BY 1 ORDER BY 1;
```

Score replay (the decisive one — no score exists in the data):

```sql
WITH RECURSIVE o AS (
  SELECT responder, sent_at, outcome,
    row_number() OVER (PARTITION BY responder ORDER BY sent_at) AS rn
  FROM pings WHERE responder = 'Vesper'
), w AS (
  SELECT responder, sent_at, outcome, rn,
    LEAST(1.0, GREATEST(0.0, 0.5 + CASE WHEN outcome='taken' THEN 0.08 ELSE -0.12 END))::numeric AS score
  FROM o WHERE rn = 1
  UNION ALL
  SELECT o.responder, o.sent_at, o.outcome, o.rn,
    LEAST(1.0, GREATEST(0.0, w.score + CASE WHEN o.outcome='taken' THEN 0.08 ELSE -0.12 END))::numeric
  FROM w JOIN o ON o.responder = w.responder AND o.rn = w.rn + 1
)
SELECT DISTINCT ON (responder, date_trunc('week', sent_at))
  responder, date_trunc('week', sent_at)::date AS wk, ROUND(score, 2)
FROM w ORDER BY responder, date_trunc('week', sent_at), sent_at DESC;
```

Counterfactual replay — what the score would be if a miss cost nothing
(section 6b). Change the `CASE` to drop the miss penalty; everything else is
identical to the replay above:

```sql
LEAST(1.0, GREATEST(0.0, prev + CASE WHEN outcome='taken' THEN 0.08
                                     WHEN outcome='turned down' THEN -0.12
                                     ELSE 0 END))
```

Credits thrown away at the ceiling — track the score *before* each ping as
well as after, then count `outcome='taken' AND prev >= 1.0`.

Ticket themes by week:

```sql
SELECT date_trunc('week', filed_at)::date AS wk,
  SUM(CASE WHEN subject ILIKE '%moved on%' OR subject ILIKE '%before he could%'
        OR subject ILIKE '%before she could%' OR subject ILIKE '%vanished%'
        OR subject ILIKE '%gone before%' OR subject ILIKE '%lost one%'
        OR subject ILIKE '%missed it%' OR subject ILIKE '%reaching for the phone%'
      THEN 1 ELSE 0 END) AS theme_a_timeout,
  SUM(CASE WHEN subject ILIKE '%hardly anything%' OR subject ILIKE '%nothing coming through%'
        OR subject ILIKE '%not gone off%' OR subject ILIKE '%still on the list%'
        OR subject ILIKE '%quieter than usual%' OR subject ILIKE '%very quiet%'
        OR subject ILIKE '%pass this on%'
      THEN 1 ELSE 0 END) AS theme_b_silence,
  COUNT(*) AS total
FROM support_tickets GROUP BY 1 ORDER BY 1;
```

Harm vs complaint, joined to handler and area:

```sql
SELECT r.name, r.handler, r.area,
  SUM(CASE WHEN p.sent_at < TIMESTAMP '2026-08-12' THEN 1 ELSE 0 END) AS pings_before,
  SUM(CASE WHEN p.sent_at >= TIMESTAMP '2026-08-12' THEN 1 ELSE 0 END) AS pings_after,
  (SELECT COUNT(*) FROM support_tickets s WHERE s.about_responder = r.name) AS tickets
FROM responders r LEFT JOIN pings p ON p.responder = r.name
GROUP BY r.name, r.handler, r.area ORDER BY 5;
```

---

## The one-line version

We shipped a change to a timer without instrumenting whether the thing being
timed ever arrived, and we fed its output into a score that can only go down.
Four of our best responders were removed from service by arithmetic, nobody
decided it, nobody was told, and the two who were hurt worst have handlers who
never file tickets — so we would never have found out.
