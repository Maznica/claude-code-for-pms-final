# Module 4 — Reading the Release 4.2 code

Session date: 1 November 2026
Sources: `00-rook/code/dispatch-routing/` (all five `.py` files, `README.md`,
`CHANGELOG.md`), `rook-database` connector (`pings`, `callouts`, `responders`).

This file records everything established in the Module 4 session. Modules 1–3
diagnosed 4.2 from the outside — tickets, interviews, the ping record. This
session went to the source and read the mechanism itself. It confirms the
diagnosis, **corrects one claim made in this session and one carried from
earlier**, and adds several things that are only visible in the code.

---

## 1. The headline

**The service that decides who gets pinged is 232 lines across five files. It
contains four `if` statements. Not one of them looks at who the responder is.**

That single fact answers Marcus's 14 August question outright, and it took
about ten minutes to establish. The question sat open for eleven weeks.

The second headline, from Lab C: **exactly one line in the entire service adds
points to a responder's score, and it can only run when a ping is answered.**
There is no reset, no decay, and no path back. A responder on the floor is
dependent for recovery on the one input their floored score prevents them from
receiving.

---

## 2. The architecture, in plain English

### The flow, end to end

| Step | What happens | Where it lives |
| --- | --- | --- |
| 1 | A callout arrives — region, location, required tags | caller of `dispatch()` |
| 2 | Fetch everyone available | `availability.py:12` |
| 3 | Score each one, sort highest first | `routing.py:18`, `routing.py:30` |
| 4 | Push to the top phone, start a 60s clock | `offer.py:41`, `config.py:9` |
| 5 | Taken → `+0.08`, done | `offer.py:28` → `history.py:27` |
| 6 | Anything else → `−0.12`, next person | `offer.py:30` → `history.py:39` |
| 7 | Loop until someone says yes, or the list runs out | `offer.py:25` |

### The file map

| File | Lines | What it owns |
| --- | ---: | --- |
| `config.py` | 26 | Every tuning number. No logic. |
| `availability.py` | 35 | Who's free, where they are, travel time. **Mostly stubs.** |
| `routing.py` | 65 | Scores and sorts. The ranking logic. |
| `offer.py` | 63 | Pushes to the phone, runs the clock, walks the list. |
| `history.py` | 43 | The recent-acceptance score. |

### The score

```
score = 0.60 × proximity  +  0.25 × recent acceptance  +  0.15 × capability
```

`routing.py:38-42`, with the weights at `config.py:13-15`. Higher wins. The
weights are plain module-level constants read fresh on every call — one copy,
no per-responder variants, nowhere for an exception to live.

---

## 3. Proximity — a sliding scale, not a switch

`routing.py:52-54`:

```python
if minutes >= PROXIMITY_HORIZON_MINUTES:
    return 0.0
return 1.0 - (minutes / PROXIMITY_HORIZON_MINUTES)
```

Take travel time, divide by 45, subtract from 1. A straight linear slope.

| Travel time | Proximity score | × 0.60 = points |
| ---: | ---: | ---: |
| 0 min | 1.00 | 0.600 |
| 10 min | 0.78 | 0.467 |
| 22½ min | 0.50 | 0.300 |
| 30 min | 0.33 | 0.200 |
| 44 min | 0.02 | 0.013 |
| 45 min + | 0.00 | 0.000 |

Two consequences:

- **Every minute costs the same.** No "close enough" plateau — a constant drip
  of 2.2% of the proximity score per minute.
- **A genuine cliff at 45 minutes** (`config.py:26`). Someone 46 minutes out
  and someone four hours out score identically. The horizon is the only thing
  in the service that effectively excludes anyone.

### What the weight change did, in minutes

The 4.2 rebalance did not change the shape of the slope. It changed what the
slope is worth against the acceptance score. Converting the acceptance gap into
its travel-time equivalent:

| Comparison | Before 4.2 | After 4.2 |
| --- | ---: | ---: |
| Floored (0.0) vs neutral (0.5) | ≈ 20 min handicap | **≈ 9 min** |
| Floored (0.0) vs ceiling (1.0) | ≈ 40 min handicap | **≈ 19 min** |

*(0.5 × 0.25 ÷ 0.60 × 45 ≈ 9.4; 0.5 × 0.40 ÷ 0.45 × 45 ≈ 20.0)*

**The weight change reduced the penalty carried by a low scorer.** This is the
arithmetic behind the Module 1 conclusion, now stated in units a non-engineer
can picture. It is the mercy half of the release.

But note the second row: a floored responder against a ceiling responder still
carries a ~19 minute handicap after 4.2. Because every other responder was
*gaining* `+0.08` on the work the floored four were no longer being offered, the
gap widened from both ends. That is enough to flip the order permanently when a
neighbouring area's responder is within 19 minutes — which, in this city, they
generally are.

---

## 4. CORRECTION — there is no region gate

**A claim made earlier in this session was wrong and is retracted here.**

Reading `available_for(region, at)` at `availability.py:12`, I asserted that
responders outside the callout's region are filtered out before ranking. That
was inferred from a parameter name. The function body is `...` — empty. No
filtering was ever observed.

The ping record disproves it:

| | Pings | Share |
| --- | ---: | ---: |
| Responder's `area` matched the callout's | 1,269 | 75% |
| Responder's `area` did **not** match | **427** | **25%** |

And the out-of-area pings are not scrapings from the bottom of the list:

| Position in the order | Same area | Different area |
| ---: | ---: | ---: |
| 1st person asked | 975 | **344** |
| 2nd | 266 | 71 |
| 3rd | 26 | 12 |
| 4th | 2 | 0 |

**344 times the first person asked was from outside the callout's area.**

Examples, each taken on the first ping:

- Uptown, man dressed as a lighthouse → **Sgt. Falkirk** (Kingsbridge)
- Harborside, man with a very large magnet → **Ironvale** (Riverside)
- Old Town, woman on stilts on the bridge → **Nightwell** (Midtown)

### What `area` actually is

Each callout area draws from its own patch plus the same **two or three**
others, never from all fifteen:

| Callout area | Draws from |
| --- | --- |
| Eastgate | Eastgate, Midtown, Foundry Row |
| Harborside | Harborside, Southport, Riverside, Greenway |
| Uptown | Uptown, Kingsbridge, Mill District, Foundry Row |

Those are **neighbours**. `area` is a descriptive label — the database calls it
*"the part of the city this responder usually covers"*. It is not a boundary.
What keeps Westbury responders out of Eastgate callouts is the 45-minute
travel-time horizon, not a region check.

**Still open:** whether `available_for` narrows by region *at all* cannot be
determined from this code. What is certain is that if it does, it is **not** a
hard match on the responder's `area` column. Two fields that sound like the same
concept and behave differently is worth a question to Wen Li in its own right.

**This also sharpens the Module 2 finding** that out-of-area pings rose 17% →
39%. That was never a rule being broken. It is the ranking working exactly as
designed, with four local responders removed from contention.

---

## 4a. Out-of-area acceptance — unreported, and strange

Splitting every ping by whether the responder's `area` matched the callout's is
the only proximity signal the data supports. It is two buckets where we wanted a
continuum, but it shows something nobody at Rook has stated.

| Period | Bucket | Pings | Taken | Turned down | Missed |
| --- | --- | ---: | ---: | ---: | ---: |
| Before | Same area | 899 | **831 (92.4%)** | 62 | 6 |
| Before | Neighbouring | 186 | **0 (0%)** | 167 | 19 |
| After | Same area | 370 | **305 (82.4%)** | 27 | 38 |
| After | Neighbouring | 241 | **86 (35.7%)** | 83 | 72 |

**Before 4.2, an out-of-area ping was never once accepted — 0 of 186**, and 167
of those were active refusals, not misses. Somebody looked at their phone and
tapped no, every single time.

### It decomposes the headline drop

Acceptance fell 76.6% → 64.0%, but **neither bucket fell by anything like that**:

- Same-area acceptance: 92.4% → 82.4%, **down 10 points** — the timeout effect.
- Neighbouring-area acceptance: 0% → 35.7%, **up 36 points**.
- Neighbouring pings went from **17.1% to 39.4%** of all pings.

**The metric fell mainly because work shifted into a bucket that converts at
half the rate** — not because responders got worse. This is a mix effect, and
it is invisible in the weekly aggregate Ravi reports.

### A wrong explanation, corrected

An initial reading was that out-of-area pings were cascade overflow — the local
refuses, it rolls next door. **That is false.** Split by position:

| Out-of-area pings | Before: sent | Before: taken | After: sent | After: taken |
| --- | ---: | ---: | ---: | ---: |
| As the **first** ping | **172** | **0** | **172** | **51 (29.7%)** |
| 2nd or later | 14 | 0 | 69 | 35 |

172 of the 186 were the **top-ranked choice**. The system put an out-of-area
responder at the head of the queue 172 times before 4.2 and was refused every
time. (The identical 172 in both periods is itself an oddity.)

### Best available reading

Before 4.2, acceptance weight (0.40) nearly matched proximity (0.45), so "ranked
first" meant best *combined* score — a responder with a strong acceptance record
could outrank a local while being genuinely far, then decline because it was
far. After 4.2, proximity dominates at 0.60, so an out-of-area responder reaches
first place by actually being near a boundary, and those people accept.

If right, **the weight change improved out-of-area targeting** — another point
for 4.2's rebalance being the good half. **Untestable**, for the same reason as
everything else: it is a claim about minutes, and minutes are not stored.

### The caveat that must travel with this

A perfect 0 of 186 is the signature of a *definition*, not a behaviour. If
`responders.area` is derived from where each responder actually took callouts,
then "nobody took a callout outside their area" is true by construction and
tells us nothing. Against that: 86 out-of-area acceptances exist after 4.2.

**Unresolvable from inside the data.** One question to Ravi or Wen Li settles
it: *is `responders.area` a field somebody sets, or is it computed from callout
history?* Until answered, treat the 0% — and both charts built on this split —
as provisional.

---

## 5. Travel time is the biggest input and we cannot see a single value of it

`availability.py:21-26`:

```python
def travel_time_minutes(responder, location):
    """Estimated minutes for this responder to reach this location.

    Travel-time estimate, not straight-line distance — changed in 4.1.
    """
    ...
```

Empty. The real lookup lives in a service outside this folder.

**And there is no travel time anywhere in the database either.** `pings` has
four columns — callout number, responder, sent-at, outcome. `callouts` has an
area and a one-line description. No coordinates, no distance, no estimate.

So the input carrying **the largest weight in the whole ranking (0.60)** is
unobservable after the fact. We cannot check whether the estimates are any good,
whether the 4.1 switch from straight-line distance improved them, or whether a
skipped responder was genuinely far away or just mis-estimated.

This belongs with the instrumentation gaps already logged: `pings` has no
delivery field and no time-to-accept. **Add: no travel-time estimate recorded on
the ping.**

### The stubs mark the boundary of what code can answer

Three of five files are mostly `...`: `availability.py` (all three functions),
and `offer.py:51`, `:56`, `:61` (push, poll, withdraw). An empty body is the
code saying *the answer isn't here*. Questions that land on one need a person or
another system, not more reading. This is a reusable habit, not a one-off.

---

## 6. Lab B — Marcus's question, answered from the source

> **Marcus, 14 Aug 2:47 PM:** *"Quick one, did the change to who gets asked
> first apply to responders who'd already been turning jobs down, or just new
> ones? Can't tell from the code and don't want to guess on this."*

### The literal answer: everyone, at once, nothing grandfathered

Every conditional in the entire service:

| Where | What it asks |
| --- | --- |
| `offer.py:27` | Was the answer yes? |
| `offer.py:45` | Has the phone replied yet? |
| `routing.py:52` | Is travel time past 45 minutes? |
| `routing.py:62` | Does this callout require any capability tags? |

**None of them asks anything about who the responder is.** No tenure check, no
cohort, no special handling. `routing.py:30` runs identically for every person.

When the two weights changed on 12 August, every responder was rescored under
the new arithmetic on the very next callout, carrying whatever score they had
already accumulated. The config "not distinguishing", as Marcus put it, is not
an ambiguity to be resolved — **it is the answer.** There is nowhere for a
distinction to live.

### The premise is backwards

Marcus is worried the change *penalised* people who had been turning jobs down.
The weight on acceptance history went **down**, 0.40 → 0.25 — which shrinks how
much a bad score costs. For anyone who had been refusing, 4.2 made things
**better**.

The roster numbers agree:

| | Before 4.2 | After 4.2 |
| --- | ---: | ---: |
| Taken | 76.6% | 64.0% |
| **Turned down** | **21.1%** | **18.0%** ↓ |
| **Missed** | **2.3%** | **18.0%** ↑ |

Refusals fell three points. Acceptance collapsed anyway, because misses went up
nearly eightfold.

### The test that settles it

If "had been turning jobs down" were the risk factor, the heaviest pre-4.2
refusers should be the ones who went quiet. They are not:

| Rank by refusal rate before 4.2 | Refused | Pings in Sept |
| --- | ---: | ---: |
| 1. Meteor Mite | 28.6% | **1** 🔻 |
| 2. Halfmoon | 27.5% | 8 |
| 3. Sgt. Bulwark | 24.6% | 8 |
| 4. Captain Vantage | 23.7% | 17 |
| 5. Farlight | 22.7% | **0** 🔻 |
| 6. The Longcast | 22.2% | 8 |
| 7. Ironvale | 21.6% | 10 |
| 8. The Undertow | 21.3% | **1** 🔻 |
| 9. Nightwell | 21.1% | 20 |
| 10. Stormwrack | 20.7% | 18 |
| 11. Corporal Ashgrove | 20.6% | 7 |
| 12. Cindermark | 19.3% | 8 |
| 13. Sgt. Falkirk | 17.5% | 12 |
| 14. The Gale | 17.1% | 19 |
| **15. Vesper** | **15.1%** | **1** 🔻 |
| 16. The Drift | 12.8% | 8 |

The four who went silent sit at ranks **1, 5, 8 and 15** — scattered through the
list. The decisive pair: **Vesper refused 15.1% and was suppressed; Halfmoon
refused 27.5%, nearly double, and is fine on 8 pings.**

Swap in misses during the first five days after 4.2 and the same sixteen split
perfectly:

| Misses, 12–16 Aug | Pings in September |
| --- | --- |
| **4 or 5** — Meteor Mite, Farlight, Vesper, The Undertow | **0, 1, 1, 1** |
| **3 or fewer** — the other twelve | **7 to 20** |

Stormwrack missed 3 and got 18. Vesper missed 4 and got 1. **No overlap.**

Prior refusing predicts nothing. Early misses predict everything. This
independently reproduces the Module 3 break-even finding (60% taken rate) from
the code side rather than the data side.

### The reply to give Marcus

> It applied to everyone at once — there is no branch anywhere in the service
> that treats any responder differently, so there was nothing to grandfather.
> But the premise is the wrong way round: lowering the acceptance weight
> *reduced* the penalty on anyone who had been turning jobs down, and refusals
> actually fell after 4.2, 21.1% → 18.0%. The responders who went quiet are not
> the ones who were saying no. They are the ones who **missed** pings in the
> first days after the timeout went to 60 seconds — and because `offer.py:30`
> charges a miss exactly what it charges a refusal, and `history.py:35` never
> lets the score recover, four of them have been stuck ever since.

---

## 7. Lab C — the complete inventory of everything that touches the score

Searched all five files for every mention of the score store, both point
constants, and the words *reset*, *decay*, *restore*. The complete list:

| Line | What it does |
| --- | --- |
| `history.py:22` | **Reads** a score |
| `history.py:27` | **Adds** 0.08 |
| `history.py:39` | **Subtracts** 0.12 |
| `history.py:43` | **Writes** the result — the only writer |

Four lines. That is the whole mechanism.

### Points off — `history.py:35-39`

```python
def record_declined(responder):
    """They turned it down, or we ran out of time waiting. Score goes
    down. Same either way — we asked and we didn't get a yes.
    """
    _set(responder, recent_acceptance(responder) - DECLINE_PENALTY)
```

Look up the current score, subtract 0.12, save it back. The function name says
*declined*; the docstring says *"or we ran out of time waiting"*. One function,
two causes, identical cost.

Called from exactly one place, `offer.py:27-30`:

```python
if answer == TAKEN:
    history.record_accepted(responder)
    return responder
history.record_declined(responder)
```

Line 27 asks one question. Anything that is not a yes falls to line 30.
**`offer.py:30` is the exact line where "turned down" and "missed" stop being
different things.** `offer_to()` genuinely returns three distinct values
(`TAKEN`, `TURNED_DOWN`, `MISSED`); the distinction is discarded one line later
and never recorded anywhere.

### Points on — exactly one thing

`history.py:25-27`:

```python
def record_accepted(responder):
    """They took the callout. Score goes up."""
    _set(responder, recent_acceptance(responder) + ACCEPTANCE_CREDIT)
```

Called from exactly one place — `offer.py:28` — behind the `if answer == TAKEN`
guard.

**The only way up has a precondition the score itself controls.** To gain points
you must tap yes; to tap yes you must receive a ping; to receive a ping you must
rank high enough; to rank high enough you need points. A responder at 0.0 can do
nothing — be available, be nearby, be perfectly qualified, phone in hand — that
touches their score, because none of those things calls line 28.

### What is NOT there

- **No reset.** `_set` at line 43 is the only writer; only two functions call
  it. No admin path, no handler action, nothing in the console.
- **No decay.** `history.py:30-34` is a comment *asking* whether there should
  be one, signed `TODO(2019)`, answering *"Leaving it as-is for now."* Seven
  years of "for now". The argument recorded against decay — *"if somebody has
  stopped taking work, we probably want that to stick until they take work
  again"* — is sound until the system stops asking. Then "until they take work
  again" becomes *forever*, and a note from 2019 is doing something its author
  did not intend.
- **No `NEUTRAL_SCORE` restore.** See below.
- **No time in the calculation at all.** No timestamps, no window, nothing
  "recent" despite the name. A miss from 2019 counts exactly as much as one from
  this morning.

### `NEUTRAL_SCORE` is a default, not a reset — and it creates the only old/new asymmetry

`history.py:22`:

```python
return _scores.get(responder.name, NEUTRAL_SCORE)
```

The 0.5 fires **only while a name is absent from the dictionary** — a brand-new
responder. The instant someone gets their first outcome, line 43 puts their name
in, `.get()` always finds a real value, and the 0.5 is never reached again.

This is the one place old and new responders differ, and it predates 4.2 — the
weights apply identically to both; only the starting value differs. Consequences:

- A newcomer who has never attended a single callout starts at **0.5**, which
  **outranks all four suppressed veterans at 0.0**. The system treats *unknown*
  as better than *known to have missed four pings*.
- There is no path back to "unknown". The fresh start is available only to
  people who have never existed in the store.
- **Useful design anchor:** 0.5 is the system's own documented answer to "where
  should someone with no evidence against them sit". That is a defensible target
  for any reset tool we specify.

In our data this asymmetry has no observable effect — all 16 responders existed
before 4.2, so nobody started fresh during the window.

### The asymmetry, and the size of the hole

Off: **0.12**. On: **0.08**. Down 50% harder than up.

- One miss takes **1.5** acceptances to undo.
- The four floored responders are 0.5 below neutral — **seven acceptances** to
  climb back to where a newcomer starts, seven pings they are not being sent.

### One open technical question

`_scores = {}` at `history.py:17` is a plain in-memory dictionary. In this code
a process restart would wipe every score back to 0.5. Either production persists
it elsewhere (outside this folder), or a restart silently resets the whole
roster. The four stayed suppressed from mid-August to 6 September, so either no
restart happened in that window or it is persisted. **Worth asking Wen Li** —
if a restart does reset it, that is simultaneously an accidental fix and an
uncontrolled source of variation in every number we have analysed.

---

## 8. The working theory, stated and tested

> **Some responders are getting no pings at all because their acceptance score
> is stuck on the floor and nothing in the code can ever lift it off — the only
> way up is accepting a ping, and they are not being sent any.**

A trap with the door on the inside. The chain:

1. **The clock got shorter** — 90s → 60s, `config.py:9`.
2. **Not answering costs exactly as much as refusing** — `offer.py:30`.
3. **Four misses in the first five days was enough** — the threshold separates
   all sixteen with no overlap.
4. **The gap widens from both ends** — the four fall while everyone else climbs
   on the work they are no longer offered. Floor vs ceiling ≈ 19 minutes of
   equivalent travel time.
5. **The neighbour is within that margin** — see below.
6. **Nothing ever lets go** — no decay, no reset.

### The decisive test: the work moved next door

September callouts in the four affected areas, with who actually got pinged:

| Area | Callouts in Sept | Who got pinged |
| --- | ---: | --- |
| **Old Town** | 13 | Captain Vantage (Hillcrest), Nightwell (Midtown), The Longcast (Lakeshore), **Vesper (Old Town) once** |
| **Uptown** | 6 | Cindermark (Mill District), Sgt. Bulwark (Foundry Row), Sgt. Falkirk (Kingsbridge) — **Farlight appears nowhere** |
| **Harborside** | 10 | Ironvale (Riverside), Stormwrack (Southport), The Drift (Greenway), **The Undertow once** |
| **Eastgate** | 16 | Nightwell (Midtown), Sgt. Bulwark (Foundry Row), The Gale (Eastgate), **Meteor Mite once** |

**The work never went anywhere. It moved next door.** This is what closes the
gap between "flooring costs ~19 minutes" and "they get literally zero" — each
area has essentially one local responder plus two or three neighbours in range,
so a 19-minute handicap is more than enough to flip the order permanently.

### Why this matters more than the other theories

Every competing explanation — seasonal softness, the weight change, August being
quiet — describes something that **fixes itself** when conditions change. This
one describes a state that **persists after the cause is gone**. Restoring the
90-second timeout tomorrow would not move these four, because the thing holding
them down is no longer the timeout. It is a stored number with no upward path.

Which inverts Priya's advice. "Wait for September, don't pull the routing change
apart" is good counsel if the problem is seasonal. It is the worst available
counsel if the problem is a one-way ratchet, because waiting is the one action
that cannot help and the damage compounds throughout.

### Where this theory could still be wrong

- **The biggest input is invisible.** Proximity is 0.60 of the score and no
  travel-time figure exists in the code or the database. The chain assumes the
  four were competitive on travel time before August and that only the
  acceptance score changed. **That cannot be checked.** If their estimates also
  degraded, the pattern looks identical from here.
- **Sixteen people is a small sixteen.** A perfectly clean threshold can happen
  by luck at this sample size.
- **One ping each is not zero.** Vesper, The Undertow and Meteor Mite each got
  exactly one September ping. Worth knowing what produced them.
- **Nobody has confirmed the miss-equals-refusal rule was a decision.** The code
  documents the behaviour twice, so it was not an accident — but that is not the
  same as someone having weighed it.

---

## 8a. The recovery path — what would have to happen for a suppressed responder to get work again

Asked from the opposite direction: somebody has been quiet for a month. Step by
step, what has to occur for them to start getting work? Traced with Vesper, who
received one ping in September.

### The seven gates

| # | What must happen | Where | Realistic? |
| ---: | --- | --- | --- |
| 1 | A callout lands within 45 min of her | `routing.py:52`, `config.py:26` | Yes — Old Town had 13 in September |
| 2 | She is marked available at that moment | `availability.py:12` | Yes |
| 3 | **She out-scores everyone else in range** | `routing.py:30` | **Once in thirteen** |
| 4 | The ping reaches her phone | `offer.py:51` *(stub)* | Unverifiable — delivery is not instrumented |
| 5 | She notices and taps yes inside 60s | `offer.py:42-46`, `config.py:9` | Fails more often than it succeeds |
| 6 | The code records it: 0.0 → 0.08 | `offer.py:28` → `history.py:27` | Automatic |
| 7 | **Repeat six more times without a miss in between** | — | **See below** |

At step 3 her score is `0.60 × proximity + 0.25 × 0.0 + 0.15 × capability`. **The
middle term contributes nothing.** She competes on two components while her
neighbours collect the full 0.25, so she must be roughly **19 minutes closer**
than they are, or they must be unavailable, out of range, or missing a tag.

Step 6 is worth putting in the travel-time units: one accepted ping buys back
**1.5 minutes** of equivalent proximity, out of the ~19 she needs.

### The basin — the key structural finding

Step 7 is the one that kills it, because the clamp at `history.py:43` makes the
arithmetic change shape depending on where she is standing:

| Her score | A yes | A miss or refusal | She climbs only if her yes-rate exceeds |
| ---: | --- | --- | ---: |
| **0.00** | → 0.08 | → 0.00 *(clamped — costs nothing)* | **0%** |
| **0.08** | → 0.16 | → 0.00 *(loses the lot)* | **50%** |
| **0.12 +** | +0.08 | −0.12 | **60%** |

At the floor she cannot fall, so any yes is free progress. The moment she steps
off it, the full −0.12 returns and she needs a yes-rate above **60%** merely to
hold station. That 60% is `0.12 ÷ (0.08 + 0.12)` — the same break-even
established from the data in Module 3, arrived at here from the code.

**Her current yes-rate is roughly 35–45%** (Module 2: the four miss 53–64% of
the pings they still get, against 10–18% for everyone else). Expected drift
above the floor at that rate is about **−0.04 per ping**.

**So she is not at the bottom of a pit. She is in a basin of attraction.** She
can bounce between 0.00 and 0.08 indefinitely — a lucky ping lifts her, the next
miss drops her straight back — and she cannot escape at her current rate.

The trap closes on itself: **her yes-rate is low *because* the pings stopped.**
She is not watching a phone that does not ring. The one condition required for
escape is precisely the thing suppression destroyed.

### What would actually have to happen

Nothing a responder or a handler can do. Every realistic route requires someone
at Rook to change something:

1. **Someone writes to the score directly.** No such tool exists — `history.py:43`
   is the only writer and only two functions call it. Smallest fix. **0.5 is its
   natural target**, being the system's own answer to "where should someone with
   no evidence against them sit".
2. **A restart, if `_scores` is not persisted.** `history.py:17` is a plain
   in-memory dict. If that is genuinely production behaviour, a deploy resets the
   whole roster to 0.5 and the problem evaporates — and every score figure in
   Modules 3 and 4 becomes fragile. **Cheapest question on the list; ask first.**
3. **A code change** — decay toward neutral (the deferred `TODO(2019)`),
   separating a miss from a refusal at `offer.py:30`, or both.

### The consequence to raise before the timeout fix ships

Restoring the 90-second timeout stops new people falling in. **It does nothing
for the four already in the basin** — they would still need a yes-rate above 60%
to climb out, and they are nowhere near it.

So when the timeout fix lands and the acceptance metric recovers, these four will
still be getting nothing, and the headline number will report the problem solved.
That belongs in front of Marcus **before** the fix ships, not after.

---

## 8b. "Nobody came" — the anatomy, and the length of the list

### Where it lives

**In the code:** `offer.py:19-31`. `dispatch()` *"Returns the responder who took
it, or None if we got to the bottom of the list without a yes."* That `None` is
the only representation of a coverage failure.

**In the database: nowhere.** No column, no flag. It must be derived — a callout
with at least one ping and no `taken` among them. Derived that way it reproduces
the Module 1 figure exactly: **48 of 879 before (5.5%), 49 of 440 after
(11.1%)**. No callout received zero pings, so every incident reached somebody.

### The rise is entirely misses

| Cause of failure | Before | After | Rate per callout |
| --- | ---: | ---: | --- |
| Refusals only (no miss involved) | 45 | 22 | **5.1% → 5.0%** |
| At least one miss involved | 3 | 27 | **0.3% → 6.1%** |

**The refusal-driven failure rate did not move** — 5.1% to 5.0%. Every bit of
the doubling is missed pings. Callouts killed by a single unanswered ping went
from **3 to 21** against half the callout volume.

Same cause as the acceptance drop and the suppression: the 60-second window.
Third symptom, one disease.

### Every failed callout, by shape

| Pattern (N = turned down, M = missed) | Before | After |
| --- | ---: | ---: |
| `N` — one refusal, callout abandoned | 39 | 19 |
| `M` — one unanswered ping, callout abandoned | 3 | **21** |
| `NN` | 6 | 2 |
| `MN` / `NM` | 0 | 6 |
| `NNN` | 0 | 1 |
| **Total** | **48** | **49** |

### The finding underneath: the list is one person long

**82 of the 97 failed callouts died after a single ping.** Average pings on a
failed callout: **1.13 before, 1.20 after**. The deepest cascade in the whole
dataset is 4.

This contradicts the source. `offer.py:25` loops the full ranked list;
`routing.py:21` says *"Nobody is removed here"*; the README says *"Everyone
available is on the list."* **The data says the list is usually one name.**

### The decisive logical test

If `dispatch()` walked to the end, a successful callout stops at the first yes
while a failed one must have exhausted everybody — so **failures should have the
longest cascades**. They have the shortest:

| | Avg pings | Max |
| --- | ---: | ---: |
| Somebody came (before) | 1.24 | 3 |
| Nobody came (before) | **1.13** | 2 |
| Somebody came (after) | 1.41 | 4 |
| Nobody came (after) | **1.20** | 3 |

Logically impossible if the whole list is walked. Consistent with the list
genuinely being shorter on those callouts — fewer available, fewer asked, more
likely to fail.

### CORRECTION — the 45-minute horizon does not exclude anyone

An earlier claim in this session that responders past the horizon are dropped
from the list is **wrong and retracted**. `routing.py:45-54` returns a *score* of
`0.0`; the responder stays in `scored`, stays in the sort, and is returned by
`rank_for_callout`. **`available_for` is the only thing in the service that can
shorten the list.** (Second instance of the same error — see section 10.)

### Testing why the list is short

Three tests for the availability hypothesis. **All three came back null.**

1. **Time of day** — failure rate by band: 00–05 **5.5%**, 06–11 7.0%, 12–17
   **8.5%**, 18–23 6.9%. Night performs *best*. No availability signal.
2. **Concurrency** — other responders with a ping within ±45 min: **1.77 on
   failed callouts, 1.77 on successful ones.** Identical.
3. **Pool size vs asked** — responders who have ever covered each area against
   how many were asked before giving up:

| Area | Plausible pool | Asked before abandoning |
| --- | ---: | ---: |
| Eastgate | 4 | 1.08 |
| Westbury | 3 | **1.00** |
| Riverside | 3 | **1.00** |
| Kingsbridge | 3 | 1.20 |
| Midtown | 5 | 1.50 |

Westbury and Riverside **never once went past the first person**.

### Three explanations, none separable from here

1. **Availability is genuinely that restrictive** — of three or four plausible
   candidates, usually one is marked on. Plausible for independent responders
   who set their own windows.
2. **Production does not do what this code says.** `available_for` is a stub; the
   real implementation may narrow by region, cap the list, or stop after N
   offers, and this folder would never show it.
3. **The pool is pruned somewhere unseen** — by the caller of `dispatch()`, or
   inside the availability service.

All three produce the observed pattern. **Availability is not recorded anywhere
— no column, no history, no timestamp** — so the database cannot choose between
them. Fourth instrumentation gap, and the one that matters most here.

### Why this changes the stakes

If the candidate pool is routinely one or two deep, removing four responders
from contention is not a ranking nuisance — it is a meaningful share of
coverage, and the Q4 "shared cover between responders" exploration stops being a
nice-to-have.

**Top question for Wen Li, ahead of the score-reset one:**

> When a callout fails, how many responders were actually on the list? We see an
> average of 1.16 pings on failed callouts and the code says it walks the whole
> ranked list — those cannot both be true.

Either way the ask is the same: **log the candidate list size on the callout.**
One integer turns a three-way guess into a fact.

### A vocabulary problem on top of a coverage problem

**"Nobody came" reads as *every responder refused*. It means *the one person we
asked refused*.** Handlers, Nadia and Ravi almost certainly read it the first
way. Per Module 1, the metric is referenced nowhere at Rook, so nobody has had
cause to notice.

---

## 8c. The what-if simulator

Built this session: a client-side replay of all 1,696 pings, 29 Jun – 6 Sep,
with adjustable rules. Four lines for the suppressed responders, a band showing
lowest-to-median for the other twelve, a marked release line at 12 August.

### What the y-axis is — read this before quoting any number off it

**The chart plots `recent_acceptance` only — one of three ranking inputs,
weighted 0.25.** It is NOT the routing score.

```
routing score = 0.60 × proximity          ← needs travel time. NOT COMPUTABLE.
              + 0.25 × recent_acceptance   ← what the simulator plots
              + 0.15 × capability_match    ← needs tags. NOT COMPUTABLE.
```

Only the middle term is derivable, because it is fully determined by the ping
sequence (`0.5` start, `+0.08`, `−0.12`, clamp 0–1) and `pings` holds exactly
that. **Nobody at Rook can compute the full score**, simulator or not.

So a line falling 1.0 → 0.0 does not mean "rank collapsed to zero". It means the
0.25-weighted term swung from maximum to minimum — a 0.25 move in total score,
worth roughly **19 minutes of equivalent travel time** (section 3). Decisive
when a neighbour is ten minutes away, which is the whole case. But a component,
not a rank.

It is also **not pings received**, and **not a rank against anyone**.

Same scoping as Module 3, which calls its figures a reconstruction of ping
outcomes rather than a score.

### The four scenarios

| # | Control | Status |
| ---: | --- | --- |
| 1 | Current rules — penalty 0.12, no recovery, 60s | **Exact replay** of real arithmetic |
| 2 | Miss penalty slider, 0 → 0.12 | **Exact** — changes only what a miss costs |
| 3 | Idle recovery to a target, 0.5 → 1.0 | **Exact** — `+0.02` per ping-free day toward target |
| 4 | Restore 90s timeout | **Modelled — assumption, not measurement** |

Scenario 3 is the deferred `TODO(2019)` decay, implemented. It applies to every
responder but only bites for idle ones.

### Scenario 4's assumptions, stated

No record exists of what a missed ping would have become with thirty more
seconds — that counterfactual is nowhere in the data. The toggle therefore
assumes:

- **87% of post-4.2 misses** would have been answered in time (derived from the
  miss rate before, 2.3%, against after, 18.0% — the excess attributed to the
  timeout).
- Recovered pings split **78% taken / 22% turned down**, matching the pre-4.2
  ratio of acceptances to refusals.
- Conversion is deterministic so the chart is stable across toggles.

Both figures are defensible; neither is measured. **Treat scenario 4 as
illustrative and 1–3 as computed.**

### The limitation binding all four

**The ping sequence is fixed.** In reality, changing any rule changes who ranks
where, which changes who gets pinged, which changes everything downstream. The
simulator cannot re-rank — that needs travel time.

So every line answers *"what would their score have been given the pings they
actually received"*, which **understates recovery**: a responder whose score
rises would also start receiving more pings, compounding upward. **The recovery
scenarios are conservative.** Same caveat Module 3 recorded at its line 590.

### What it demonstrates

**Set the miss penalty to 0, change nothing else.** All four stay clear of the
floor. This isolates `offer.py:30` as the single line that did the damage — no
timeout change, no recovery mechanism, just not charging a miss like a refusal.

**Restore 90s instead, penalty back at 0.12.** All four lines improve and all
four still bottom out — section 8a's conclusion, made visible. The timeout fix
stops new people falling in and does not retrieve those already down.

### Note on the band

Drawn from the **twelve** others, not eight: sixteen on the roster minus the
four suppressed. Module 3 established Ashgrove and Halfmoon are not suppressed,
so they belong in the band.

---

## 9. What this session changes

### Confirmed from a third angle

Modules 1 and 3 diagnosed this from the ping record; Module 2 from tickets and
interviews. Module 4 reads the mechanism directly and finds the same thing. The
timeout cut, the miss/refusal collapse at `offer.py:30`, and the absent decay at
`history.py:35` are now established in the source, not inferred from behaviour.

### Corrections

1. **There is no demonstrable region gate.** Retracts a claim made earlier in
   this session. 25% of pings cross areas; 344 as the first ping. `area` is a
   descriptive label, not a boundary.
2. **The Module 2 out-of-area rise (17% → 39%) is not a rule being broken.** It
   is the ranking behaving as designed with four local responders removed.
3. **The 45-minute horizon does not remove anyone from the list.** It sets
   proximity to 0.0; the responder stays in the ranking. Retracts a claim made
   earlier in this session. `available_for` is the only filter.
4. **Out-of-area pings were not cascade overflow.** 172 of the 186 pre-4.2
   out-of-area pings were the *first* ping, not a fallback.

### New, from the code only

- The whole service has **four conditionals**, none per-responder — the
  decisive code answer to Marcus.
- Proximity is **linear, not binary**, with a hard cliff at 45 minutes.
- The weight change expressed in travel-time minutes: a floored responder's
  handicap fell from ~20 min to ~9 min against neutral.
- **Exactly one line adds points**, gated on receiving a ping.
- **No timestamps anywhere** — "recent acceptance" is not recent.
- `NEUTRAL_SCORE` 0.5 is a default, not a reset, and a newcomer outranks a
  floored veteran.
- `_scores` is an **in-memory dict** — persistence is an open question.
- **The floor is a basin, not a pit.** The clamp makes the break-even yes-rate
  vary by height: 0% at the floor, 50% at 0.08, **60% at 0.12 and above**. A
  suppressed responder can bounce between 0.00 and 0.08 forever without
  escaping. Derived from the code; matches the Module 3 figure derived from the
  data.
- **One accepted ping buys back 1.5 minutes** of equivalent travel time, against
  a ~19 minute gap to a ceiling-scoring rival.
- **The timeout fix will not recover the four.** It stops new people falling in
  while the headline metric recovers around them.
- **The candidate list is usually one name.** 82 of 97 failed callouts died
  after a single ping; failures have *shorter* cascades than successes, which is
  impossible if the full list is walked.
- **The "nobody came" doubling is entirely misses.** Refusal-driven failures
  held at 5.1% → 5.0%; single-miss failures went 3 → 21.
- **Out-of-area acceptance went 0% → 35.7%**, and the headline acceptance drop
  is largely a *mix* effect — work moved into a bucket converting at half the
  rate. Neither bucket fell by anything like 12.6 points.
- **"Nobody came" is a misleading name** — it means the one person asked
  refused, not that everyone did.

### Questions for Wen Li, sharpened

In priority order. The first three change what the fix has to be; the rest are
hygiene.

1. **How long is the list when a callout fails?** We see 1.16 pings on failed
   callouts and `offer.py:25` says it walks the whole ranked list — those cannot
   both be true. *(Top question: if the pool is routinely one or two deep, four
   suppressed responders is a coverage problem, not a ranking nuisance.)*
2. **Is `_scores` persisted, or does a restart reset the roster?** Cheapest
   question here; the answer either hands us a free fix or invalidates every
   score figure in Modules 3 and 4.
3. **Does any score-reset tool exist?** Nothing writes a score except the two
   record functions. If there is genuinely nothing, the fix is bigger than
   restoring the timeout.
4. Was the miss-equals-refusal rule at `offer.py:30` a decision or a side
   effect? *(Marcus's 14 Aug question — the first half is now answered from the
   code; this half still needs a person.)*
5. What does `region` mean in `available_for`, and is it the same thing as the
   `area` column on a responder?
6. Is the 45-minute horizon deliberate, or another number from 2019?

**For Ravi:** is `responders.area` a field somebody sets, or is it computed from
callout history? Section 4a is provisional until this is answered.

### Carried forward unchanged

Supply servicing the four suppressed responders' gear (tested and not supported
in Module 2 for the feedback-loop version, still open for the servicing
question). Availability Confidence, committed to 4.2 and absent from the release
notes — conversation with Helen.

### Instrumentation gaps — updated list

`pings` still has no delivery field and no time-to-accept. **Now add two more:**

| Missing | Why it matters | Where it would go |
| --- | --- | --- |
| Push delivery | Cannot tell a dead phone from an ignored one | `pings` |
| Time-to-accept | The headline latency metric is uncomputable | `pings` |
| **Travel-time estimate** | **Largest input to the ranking (0.60), zero trace** | `pings` |
| **Candidate list size** | Cannot tell if "nobody came" means 1 refusal or 10 | `callouts` |
| **Availability history** | The only thing that can shorten the list, unrecorded | new table |

And **no score column anywhere** — the score lives only in `history.py`, so
every score figure in Modules 3 and 4 is a replay, not a read.

**Scoping rule for every score figure in Modules 3 and 4:** they are all
`recent_acceptance` — **one of three ranking inputs, weighted 0.25**. The
routing score itself has never been computed by anyone, because proximity
(0.60) needs travel time and capability (0.15) needs tags, and neither is
stored. Never present one of these numbers as a rank or a position. The honest
phrasing is *"their acceptance component"*, and its full 0→1 range is worth
about 19 minutes of equivalent travel time.

The whole database is 19 columns across 5 tables. Every one of the questions
above is one column away from answerable.

---

## 10. A note on method

Two habits from this session worth keeping:

**An empty function body (`...`) is the code telling you the answer is
elsewhere.** Three of five files are mostly stubs. When a question lands on one,
stop reading and go find a person or another system.

**A parameter name is not a behaviour.** The region error came from reading
`available_for(region, at)` and assuming what it did. Thirty seconds of SQL
against the ping record disproved it. Where code and data can both speak to a
question, ask both — the code says what *should* happen, the data says what
*did*.

**Scoring zero is not the same as being absent.** The horizon error was the same
mistake as the region error, made a second time in the same session: reading
`return 0.0` and hearing "excluded". A responder past 45 minutes is still on the
list, still sorted, still reachable. Both errors were caught by the student, not
by me — the useful habit is that a claim about *who is on the list* must be
traced to the line that builds the list, never inferred from a line that scores
it.

**When the code and the data disagree, that disagreement is the finding.**
`offer.py:25` says walk the whole list; the ping record says one name. Neither
source is wrong about itself. The gap between them is where the undocumented
behaviour lives, and it produced the most consequential question of the session.

---

## The one-line version

The part of Dispatch that decides who gets work contains one line that can
raise a responder's standing, it only fires when a ping is answered, and four
people cannot answer pings they are no longer sent — so the recovery path for
the only four people who need it is the one path their situation makes
unreachable.
