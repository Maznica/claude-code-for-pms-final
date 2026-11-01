# Module 2 — Handler interviews and support tickets

**Lab A** — rook-wiki, **Customer interviews** database (console redesign
research, September 2026). Four recorded calls, all run by Sofia Marino.

**Lab B** — rook-database, **support_tickets**, all 147 rows, 29 June to
7 September 2026, 12 handlers, 83 still open.

**Lab C** — the two piles compared, plus the `pings` and `callouts` tables and
the Module 1 code reading (`config.py`, `offer.py`, `history.py`).

---

# Lab A — the four handler interviews

| Handler | Responder(s) | Date |
| --- | --- | --- |
| Aunt Dot | Vesper | 3 Sep 2026 |
| Mr. Ambrose | Captain Vantage | 2 Sep 2026 |
| Kip | Meteor Mite, The Gale | 4 Sep 2026 |
| Halloran | Sgt. Bulwark | 5 Sep 2026 |

Grouped **by cause**, not by symptom. See "A note on the grouping" at the end
for why this was regrouped and what the first pass looked like.

---

## Who raised what

| | Dot | Ambrose | Kip | Halloran | Count |
| --- | :-: | :-: | :-: | :-: | :-: |
| **A1** Responder can't answer in time | ● | ● | | ● | **3** |
| **A2** Handler can't see it happening | ● | ● | | | **2** |
| **A3** Weeks with no callouts at all | ● | | ● | | **2** |
| **B1** Filter silently resets | | ● | | | **1** |
| **B2** Alerts don't say which responder | | | ● | | **1** |
| **C1** Text too small to read at a glance | ● | ● | | | **2** |
| **C2** No dark mode | | | ● | | **1** |
| **D** Supply complaints (not Dispatch) | | | | ● | **1** |

---

## A. Release 4.2 cut the answer window from 90s to 60s

The largest theme by far, and the one nobody named — all three parts below were
described by handlers who had no idea they were describing the same change.

### A1. The responder can't answer in time — 3 of 4

Dot, Ambrose, Halloran. Three households, three cities, the same scene: the
responder is physically moving toward the phone and the job is already gone.

- **Dot:** "by the time he's actually got a thumb on the screen — it's gone.
  Gone to somebody else already. And he just sort of stands there in the
  kitchen holding the phone."
- **Ambrose:** "Two weeks ago the young master was halfway into the suit —
  coat on, one boot on — when it went to somebody else."
- **Halloran:** "Bulwark mentioned a callout went to someone else before he'd
  even got his boots on, a few weeks back."

Ambrose also described the change itself, without knowing it had happened:
"there was a period where a slower-arriving response of his still landed him
the job more often than not, and lately that doesn't seem to hold the way it
used to."

Halloran only said this because Sofia asked a closing catch-all question, and
framed it as "not really a complaint."

### A2. The handler can't see it happening — 2 of 4

Dot and Ambrose. Not a separate problem — this is A1 seen from the handler's
chair. The callout goes to the responder's phone; the handler is watching a
screen that doesn't carry the one thing that matters.

- **Dot:** "maybe something that tells me too, not just him, when something's
  come in, so I'm not relying on hearing a buzz through a ceiling. I don't
  need to do anything about it. I'd just like to know."
- **Ambrose:** "I heard about it after the fact, from him, not from the
  console… I would want whatever tells me a callout is live to be harder to
  miss, not easier."

Dot does get *a* chime ("a little chime, and it's a good chime"). What she
doesn't have is a signal that a ping is live on his phone with the clock
running. The materials don't settle what the console actually announces when a
ping goes out — **this is unresolved and worth asking Sofia and Wen Li.**

### A3. Four responders fell through a threshold and stopped getting pings — 2 of 4

Dot and Kip. The same cause as A1, one step downstream, and the serious half.

- **Dot:** "there have been stretches, more than one, where the phone just
  sits there and sits there and I end up cooking for someone who's home all
  week instead of half of it."
- **Kip:** "Meteor Mite's been texting me all week asking if something's
  broken because it's so quiet… And The Gale hasn't stopped, like, at all,
  same week, same city."

**The chain:** shorter window → more missed pings → a miss is penalised
identically to a refusal (`offer.py:30`, `history.py:35`) → the score never
decays (`TODO(2019)`) → fewer pings → fewer chances to earn the score back.

**It is a threshold, not a slope.** All 16 responders, misses 12–16 Aug vs
pings received in September:

| Misses 12–16 Aug | Responders | Pings in September |
| --- | --- | --- |
| 5 | The Undertow | 1 |
| 4 | Farlight, **Meteor Mite**, **Vesper** | 0, 1, 1 |
| 3 | Stormwrack | **18** |
| 2 | Halfmoon, The Drift, **Sgt. Bulwark**, Sgt. Falkirk, **The Gale**, Nightwell | 8–20 |
| 1 | Corporal Ashgrove, The Longcast, Cindermark, Ironvale, **Captain Vantage** | 7–17 |

Three misses costs nothing. Four costs everything. Nobody sits in between.

**The two handlers who described quiet weeks handle the two responders at four
misses.** Dot handles Vesper; Kip handles Meteor Mite. Kip's other responder,
The Gale — same city, same handler, two misses — got 19 pings in September.
Kip was looking straight at the mechanism: "two cards on the same screen that
might as well be two different products."

**Once through the threshold they slide, and don't recover.** Weekly pings:

| Responder | 12 Aug | 19 Aug | 26 Aug | 2 Sep |
| --- | --- | --- | --- | --- |
| Vesper | 10 | 5 | 2 | 0 |
| Farlight | 7 | 3 | 1 | 0 |
| Meteor Mite | 8 | 3 | 2 | 1 |
| The Undertow | 11 | 1 | 1 | 1 |
| The Gale (control) | 19 | 15 | 20 | 15 |
| Stormwrack (control) | 15 | 20 | 11 | 17 |

**Consequence for any fix:** restoring the window to 90 seconds fixes A1 and A2
completely and fixes A3 **not at all**. The four are already floored, and
nothing found so far decays or resets the score. Treating A1 and A3 as one item
risks shipping one config change, watching the headline metric recover, and
leaving four responders silent.

### A4. Two knock-on effects visible only in the data

Neither was raised in an interview. Both follow from the same 60-second cut.

**Callouts nobody attended doubled: 5.5% → 11.1%** (48 of 879 before 12 Aug,
49 of 440 after).

This needs no new mechanism. Healthy responders' not-taken rate went from ~23%
before to ~28–36% after, and a callout gets about 1.4 attempts. The arithmetic
reproduces the figure: 0.23 × 1.2 ≈ 5%, 0.30 × 1.4 ≈ 11%. The coverage gaps
are the timeout cut compounding one ping at a time.

**The gaps are not caused by the four suppressed responders.** If they had been
removed from the pool, their own areas would be worst hit. The opposite holds:

| Area | Responder | Nobody came, before → after |
| --- | --- | --- |
| Riverside | Ironvale (healthy) | 4.9% → **26.1%** |
| Southport | Stormwrack (healthy) | 4.4% → **22.5%** |
| Kingsbridge | Sgt. Falkirk (healthy) | 7.4% → **21.4%** |
| Eastgate | **Meteor Mite (suppressed)** | 5.1% → 10.5% |
| Old Town | **Vesper (suppressed)** | 2.8% → 5.3% |
| Harborside | **The Undertow (suppressed)** | 7.9% → 6.3% ↓ |
| Uptown | **Farlight (suppressed)** | 6.8% → 3.2% ↓ |

Two of the four suppressed responders' areas improved. Each area holds only
16–40 callouts after 12 Aug, so the exact ranking isn't defensible — the
pattern is: the worst gaps are in areas whose own responder is healthy and
busy.

**Out-of-area pings went from 17.1% to 39.4%.** Before 4.2, about one ping in
six went to a responder outside their usual area; now it is two in five. Every
responder's share roughly doubled (The Gale 7→22%, The Drift 8→43%, Captain
Vantage 22→49%).

Same cause: the local responder is pinged first, misses more often now, and
the callout moves to someone further away. The consequence is that responders
are travelling further to more callouts — slower arrival on the ground, and
**it does not show up in callout acceptance rate at all.** Nobody at Rook has
raised this.

Worth holding against the glossary, which lists mutual aid — responders
covering across areas — as "not supported today, Q4 exploration." Whatever the
formal position, 39% of pings now cross areas. A question for Wen Li, not a
conclusion.

---

## B. The console changes state without telling anyone — 2 of 4

Ambrose and Kip. One shared shape: the screen quietly stops meaning what the
handler thinks it means. Half of this is a 4.2 side effect; half predates it.

### B1. The filter silently resets — 1 of 4 (Ambrose) — caused by 4.2

Ambrose filters the console down to his responder's capability tags. Before
August he had to rebuild that filter on every visit, which he had complained
about. **4.2 shipped filter persistence and he liked it immediately** — "the
difference between opening the console and immediately being at work, versus …
five minutes of housekeeping."

Then it reverted to the default view twice, silently.

- **Ambrose:** "I didn't notice until I'd been looking at the wrong filter for
  a quarter of an hour. I'd rather it warned me the filter had reset than
  simply reset it."

The feature isn't the problem — the silence is. When he rebuilt it manually he
always knew what he was looking at. Now he has to verify it: "I'd rather trust
it than keep double-checking it." A fix that must be double-checked has not
fully landed.

Worth flagging: the handover predicted filter persistence would generate
"cosmetic noise." This isn't cosmetic — it's a trust problem in the one 4.2
change that was meant to be a straightforward win.

### B2. Alerts don't say which responder they're for — 1 of 4 (Kip) — predates 4.2

- **Kip:** "Mite and Gale both just go 'bing' and I have to actually look to
  know which one. That trips me up more than I'd like to admit at 2am."

Unrelated to timing. Would still be true at 90 seconds with zero misses. Only
bites handlers with more than one responder.

---

## C. The console assumes a desk, good light and good eyesight — 3 of 4

Dot, Ambrose, Kip. **Nothing to do with 4.2.** This is the actual answer to the
question Sofia set out to ask, and the only part of the set that is genuinely
console-redesign input.

Three handlers, three settings the console doesn't fit: a kitchen counter at
standing height, a study belonging to someone whose eyes have changed, and a
three-monitor desk at 11pm.

### C1. Text too small to read at a glance — 2 of 4

Both named it as their one-thing-I'd-change.

- **Dot:** "the text is small enough that from a normal standing distance I
  sometimes can't tell if something's changed unless I lean right in, and
  leaning right in with wet hands over a laptop is not something I recommend
  to anyone."
- **Ambrose:** "at a glance I sometimes cannot tell engaged from available
  without leaning in."

### C2. No dark mode — 1 of 4

Raised four separate times in a 29-minute call, and his answer to "if you got
exactly one thing."

- **Kip:** "It's 11pm… the console is just this wall of white light in the
  middle of my face. I've tried the OS-level color filter trick, it makes the
  whole thing look sick, like everything's got a fever."

Also from Ambrose, same family, lower priority: the capability tag legend sits
in a drawer he has to go hunting for every few weeks.

---

## D. Supply, not Dispatch — 1 of 4

Halloran. A different product, no connection to 4.2, and not Dispatch's to fix
— but recorded here because he opened and closed the call with it.

- **One queue for everything.** A cracked vest plate waited eleven days on a
  signature. There is a priority field; setting it changes nothing. "One
  queue, everything in it, doesn't matter if it's laces or plate."
- **Failure reports go nowhere.** "I file a failure report and it goes into a
  void. I don't know if anyone reads it, I don't know if it changed anything."
- **Catalog search needs the exact item name.** Typing "plate" returned forty
  results, half unrelated.
- **Closing line:** "Get someone to fix the requisition queue."

He also volunteered that maintenance scheduling has improved — "it's gotten
smarter about not booking maintenance into a week he's likely to be out."

**One open thread back to Dispatch.** Supply's maintenance scheduling reads the
Responder Availability Record, which Dispatch writes and Supply only reads. The
four suppressed responders look quiet to Dispatch, so they may look freshly
available to Supply's scheduler. Whether their gear is being serviced during
artificially quiet weeks is **not checkable from the Dispatch database.**

**A hypothesis tested and not supported.** It is tempting to close the loop the
other way: suppressed responder looks idle → Supply takes the gear → responder
drops out of routing entirely → coverage gaps rise. Two things argue against it.

- *Mechanically*, the integration is one-way. Dispatch writes the availability
  record and Supply only reads it; Supply has no path to mark anyone
  unavailable in Dispatch. The loop would need a human step — a handler
  standing their responder down because the gear is away.
- *Empirically*, the coverage gaps land in the wrong places (see A4), and the
  doubling is already fully explained by the arithmetic of the timeout cut. The
  suppressed four were still pinged 11–17 times after 12 Aug, so they were not
  removed from the pool.

Not refuted — we cannot see Supply — but not needed to explain anything
observed. The Supply question is worth asking as a **harm to those four**
(gear pulled during weeks they only look idle), not as a cause of the gaps.

---

# Lab B — the 147 support tickets

Every row counted, including the ones that look unrelated. Nine groups, summing
to 147.

| # | Group | Tickets |
| --- | --- | ---: |
| 1 | Console usability and display | **30** |
| 2 | "The phone has gone quiet" | **30** |
| 3 | Supply — gear, requisitions, maintenance | **26** |
| 4 | Filter persistence | **16** |
| 5 | "The callout vanished before we could answer" | **15** |
| 6 | Account, access and admin | **13** |
| 7 | Handler workflow requests | **9** |
| 8 | Duplicate or unreliable notifications | **4** |
| 9 | Performance and export bugs | **3** |
| | Plus one pre-release missed ping (#3014) | **1** |
| | **Total** | **147** |

**1 — Console usability and display (30).** Small text, dark mode, layout,
sorting, logouts, screen reader support, the capability tag legend, the alert
sound. Dark mode alone is 4 tickets, status badge text 2, screen reader 2.

> *#3013:* "The engaged and available badges are hard to tell apart unless I
> lean right in to the screen."

**2 — "The phone has gone quiet" (30).** The largest single complaint in the
system. Every one still open.

> *#3120:* "Farlight asked me to pass this on in her own words: 'nothing again
> this week. starting to wonder if im still even in the system.'"

**3 — Supply (26).** Requisition queue, failure reports into the void, catalog
search, back orders, wrong sizes, maintenance timing. Not Dispatch's, and not
4.2-related.

> *#3063:* "I put in a requisition for a replacement grapple line… nine days
> ago and it is still waiting for a quartermaster to sign it."

**4 — Filter persistence (16).** Two requests *for* it before the release
(July), then 14 after — of which 2 are thank-yous, 4 are the silent reset, and
the rest are scope problems (shared computers, two machines, no clear-all,
badge too subtle).

> *#3044:* "Twice now my saved filters have been wiped overnight. I assume
> there was an update. I would rather it told me than quietly reset."

**5 — "The callout vanished before we could answer" (15).** Every one filed on
or after 12 August. Every one still open.

> *#3043 (Mr. Ambrose):* "Captain Vantage was in the process of suiting up,
> coat and one boot on, when it moved along to someone else… I would ask that
> whoever reviews this at least confirm the timing is what it is meant to be,
> since it felt very fast."

**6 — Account, access and admin (13).** Phone number validation, profile
photos, billing duplicates, password reset delays, second-handler sign-ins,
timezone on availability windows.

> *#3059:* "The profile page will not let me save the new one; it says the
> format is wrong, but it is a normal number."

**7 — Handler workflow requests (9).** Handover notes, on-leave status, history
export, training material, bulk callout.

> *#3035:* "There is no way to show that in the console except setting her
> unavailable, which looks the same as being busy."

**8 — Duplicate or unreliable notifications (4).** All pre-dating 4.2, all
closed.

> *#3016:* "Nightwell's phone buzzed twice for the same job last night. The
> second one showed up after she had already said yes."

**9 — Performance and export bugs (3).** Monday-morning slowness twice, and a
timezone error on the coverage export.

> *#3045:* "Every Monday between about 8 and 9 the console takes a long time to
> load the coverage view."

**The one that doesn't fit (1).** #3014, 14 July — a missed ping, but weeks
before the release and explained on its face: *"The Undertow was in a spot with
no signal last night."* The only missed-ping ticket in the dataset that isn't
4.2.

## What the ticket set says beyond the counts

**Nadia's estimate was right about volume — and backwards about prevalence.**
She reported roughly two thirds "my phone never goes off" and one third "it
buzzes and it's gone." The ticket split is 30 to 15, precisely 2:1, so her read
of the queue was accurate. But counted by *people*:

| | Tickets | Handlers |
| --- | ---: | ---: |
| Gone quiet | **30** | **4 of 12** |
| Callout vanished | 15 | **11 of 12** |

Nearly every handler experienced the vanishing; only a third experienced the
silence. Four distressed handlers filing eight times each outvoted eleven
handlers mentioning it once. "Two thirds of our problem is silence" would send
us to fix the rarer thing first. See Lab C.

**The five-day lag is the mechanism, visible in the queue.** Tickets per week:

| Week | Vanished | Gone quiet |
| --- | ---: | ---: |
| 12–18 Aug | 7 | 1 |
| 19–25 Aug | 4 | 9 |
| 26 Aug–1 Sep | 3 | 12 |
| 2–7 Sep | 1 | 8 |

"Vanished" starts on release day, peaks immediately, then falls away. "Gone
quiet" starts 17 August and climbs. Misses first, suppression after. The
"vanished" line falls partly *because* the suppression worked — you cannot miss
a ping you never receive.

**Nothing has been closed on either theme.** All 15 "vanished" and all 30
"quiet" tickets are open: 45 of the 83 open tickets in the system. Support has
resolved nothing on this since 12 August.

**The blind spot is now measurable.** Group 2 names only four responders:
Farlight, The Undertow, Corporal Ashgrove, Halfmoon. The four actually
suppressed are Farlight, The Undertow, **Vesper** and **Meteor Mite** — and
Vesper and Meteor Mite appear in **zero tickets of any kind**, because Aunt Dot
and Kip do not file them. Meanwhile Ashgrove and Halfmoon, who generate 11
tickets between them, are not suppressed — they are reporting seven pings
against a usual ten, which is the general drop in callout volume.

The ticket queue is wrong in both directions: it misses half the genuinely
harmed and adds two who are not. Triage by ticket volume and you investigate
the wrong four people.

Mr. Ambrose filed exactly one ticket in the entire 147 — #3043, the most
carefully written one in the set. Aunt Dot, Kip and Halloran filed none.

**Caveat on reading these counts.** Many bodies are verbatim duplicates across
different handlers — "Ping moved on while X was reaching for the phone" appears
four times word for word, "it vanished before he could swipe" three times. The
15 "vanished" tickets are about 5 distinct phrasings across 13 responders.
Ticket counts measure *reporting behaviour*, not distinct incidents — which is
why 147 tickets pointed everyone at the wrong half of the release.

## Why groups 2 and 5 are not the same problem

They have the same cause and feel identical. They bite at different steps of
the loop, and only one of them is fixed by putting the timer back.

**"Vanished"** — the phone rang and she could not get to it in time.
**"Gone quiet"** — the phone never rang at all.

One is losing a race. The other is not being invited to the race.

### The loop

1. A callout arrives at Rook.
2. **Rook ranks everyone available** — who gets asked first, second, third.
   Position depends on travel time, availability, tags, and recent acceptance.
3. Rook pings the top phone. One responder, not a broadcast.
4. **The responder has 60 seconds.** Yes, no, or time runs out.
5. The outcome updates their score. A miss is recorded as a refusal, and the
   score never comes back up.
6. That score feeds back into step 2.

| | "Vanished" | "Gone quiet" |
| --- | --- | --- |
| Where it bites | Step 4 | Step 2 |
| Did a ping exist? | Yes | No |
| In the database | A row with outcome `missed` | Nothing. An absence. |
| Who it happens to | Everyone — 13 of 16 responders | Only those past the threshold |
| Shape | An event: happens, ends, survivable | A state: persists day after day |
| Noticed | Instantly — the phone buzzed | After days of silence |
| Fixed by restoring 90s? | Yes, completely | **No. Not at all.** |

**The connection:** step 4 feeds step 5 feeds step 2. Every "vanished" event
pushes the responder further down the list. Enough of them and step 3 stops
happening at all. The first problem *manufactures* the second. They are the top
and the bottom of one staircase — Captain Vantage is on the top step, one miss,
still busy; Farlight fell down the whole flight.

**The handler cannot tell them apart**, because step 2 is invisible outside
Rook. Linda Pruitt can see that Farlight is not working. She cannot see the
ranking. That is why eleven of her tickets ask Rook to check *their* side.

### The same thing in plain terms — Rook, Linda, Farlight

Rook is the teacher who picks who plays. Linda is the coach who looks after one
player. Farlight is the player, boots on, waiting.

The teacher has a clipboard with every name on it, in order. When a game needs
someone she reads the name at the **top**, calls it, and the player must shout
"yes" before time runs out. If they do not, she crosses them off and calls the
next name. One more rule, which nobody tells the players: **every time you do
not shout in time, your name moves down the list, and nobody ever moves it back
up.**

**Problem one — the race got shorter.** In August the teacher quietly cut the
time from ninety seconds to sixty, and announced nothing. Farlight hears her
name, grabs her boots, runs — and it has already gone to someone else. Unfair,
but her name is still near the top. Tomorrow she will be called again.

**Problem two — she fell off the list.** In the five days after the change that
happened to her **four times**. Each one moved her name down. After four, her
name sat at the bottom, and the teacher never reads that far — someone higher
always shouts yes first. So Farlight stands in the playground with her boots on
and her name is never called. Nothing to run for, nothing to lose, just
silence. And she does not know why, because nobody told her the clipboard
exists.

In problem one she **hears her name and cannot get there.** In problem two
**there is no name to hear.** A thousand seconds to answer would change
nothing, because nobody is calling her.

**Linda stands at the edge of the playground.** She cannot see the clipboard —
nobody outside Rook can. For three weeks she asks the same question eleven
different ways, and all eleven tickets are still open.

> *#3103:* "She has started asking me every day whether something is broken. I
> do not know what to tell her."

**The night both problems happened at once.** On 29 August, after five silent
days, Farlight's phone finally went off — and the sixty-second rule took it
away again. Linda filed it as a new problem:

> *#3110:* "Separate from my earlier tickets about the quiet stretch. Something
> finally came in for Farlight last night, the first in five days, and it was
> gone before she could answer. Given how long she had been waiting, this
> landed badly."

It is not separate. It is one night where she was barely on the list **and**
lost the one race she got.

**Why keeping them apart changes what we do.** Treat them as one problem and
you put the timer back to 90 seconds, the "vanished" tickets stop, the headline
metric recovers, and the ticket closes. Farlight's name is still at the bottom
of the clipboard, and nothing in the system moves it up. She stays silent, and
so do Vesper, Meteor Mite and The Undertow — two of whom never filed a ticket
at all, so nobody is watching for them.

## Cross-cuts: by handler, by responder, by area

### Tickets by handler

| Handler | Responder | Total | Before | After | Quiet | Vanished | Filters | Supply | Other |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Desmond Okafor | The Undertow ⚠ | **21** | 4 | 17 | 10 | 2 | 1 | 4 | 4 |
| Linda Pruitt | Farlight ⚠ | **19** | 3 | 16 | 10 | 1 | 1 | 3 | 4 |
| Simone Fischer | Halfmoon | 15 | 3 | 12 | 5 | 1 | 1 | 1 | 7 |
| Yusuf Demir | Corporal Ashgrove | 14 | 4 | 10 | 5 | 0 | 1 | 2 | 6 |
| Marjorie Sung | Nightwell | 12 | 4 | 8 | 0 | 2 | 2 | 4 | 4 |
| Teresa Alvarez | Ironvale | 12 | 4 | 8 | 0 | 2 | 2 | 2 | 6 |
| Owen Bramwell | Sgt. Falkirk | 11 | 3 | 8 | 0 | 2 | 1 | 3 | 5 |
| Beatrice Calloway | The Drift | 11 | 4 | 7 | 0 | 1 | 1 | 3 | 6 |
| Graham Petrov | The Longcast | 11 | 4 | 7 | 0 | 1 | 2 | 2 | 6 |
| Farid Haddad | Cindermark | 10 | 4 | 6 | 0 | 2 | 2 | 2 | 4 |
| Renata Kovač | Stormwrack | 10 | 3 | 7 | 0 | 1 | 2 | 0 | 7 |
| Mr. Ambrose | Captain Vantage | **1** | 0 | 1 | 0 | 1 | 0 | 0 | 0 |
| **Aunt Dot** | **Vesper ⚠** | **0** | — | — | — | — | — | — | — |
| **Kip** | **Meteor Mite ⚠, The Gale** | **0** | — | — | — | — | — | — | — |
| **Halloran** | **Sgt. Bulwark** | **0** | — | — | — | — | — | — | — |

⚠ = suppressed responder.

All 30 Quiet tickets come from four handlers; Okafor and Pruitt filed 20 of
them between them. Okafor filed 21 tickets and Ambrose filed one — a 21:1 ratio
that reflects temperament, not harm. This compounds the duplicate-text caveat
above: ticket volume is a measure of who writes in, nothing more.

### Pings and tickets side by side — a second clean cliff

Outcomes after 12 August separate into two populations with nothing between:

| | Pings after 12 Aug | Missed | Miss rate | Tickets |
| --- | ---: | ---: | ---: | ---: |
| Twelve healthy responders | 28–70 | 4–9 | **10–18%** | 0–15 |
| Vesper ⚠ | 17 | 9 | **53%** | **0** |
| Meteor Mite ⚠ | 14 | 8 | **57%** | **0** |
| The Undertow ⚠ | 14 | 9 | **64%** | 21 |
| Farlight ⚠ | 11 | 7 | **64%** | 19 |

Before 12 August every responder missed 0–5% of pings.

**The four do not merely receive fewer pings — they fail most of the ones they
do receive.** Suppression alone does not predict that. A hypothesis worth
testing: after days of silence they have stopped watching the phone. Tickets
#3110 and #3140 describe exactly this — "the first in five days, and it was
gone before she could answer." If that holds, suppression and disengagement are
feeding each other, and the window for a cheap fix is closing: the longer the
four stay quiet, the worse they will perform when pings return.

**The ticket column is the alarm failing.** The Gale took 48 jobs and generated
zero tickets, which is correct — nothing is wrong. Vesper and Meteor Mite are
the two worst-affected responders in the company and also generated zero. The
signal and the silence are indistinguishable from support's side.

### By area, before and after

With one responder per area (bar Eastgate), this is largely the handler view
relabelled. It earns its place for one reason.

Before 12 August the map is flat: every area filed three or four tickets, and
**not one was about a callout.** After, it is lumpy — Harborside (17) and
Uptown (16) dominate, mostly Quiet.

**Old Town, Eastgate and Foundry Row filed nothing in either period.** Old Town
is Vesper. Eastgate is Meteor Mite. Half the suppressed population lives in
areas that look, on this map, entirely untouched. Anyone asking "where is 4.2
hurting us" from ticket data alone would be sent to Harborside and Uptown, and
would have no reason to look at Old Town or Eastgate at all.

---

# Lab C — both piles together

## The headline: they converge

Of the four handlers Sofia interviewed, **three have never filed a ticket** —
Aunt Dot, Kip and Halloran appear nowhere in 147 rows. Mr. Ambrose appears
once. The two piles are almost entirely different people.

And they tell the same story anyway. Both independently surface the same three
release problems — callouts vanishing, phones going quiet, filter persistence —
and the same console complaints. That is not two sources disagreeing; it is
replication across non-overlapping populations, and it is the strongest
validity argument in this document. If the four had told a different story from
the 147, both would be suspect.

## Where the channels differ

The problem set is shared. What differs is which parts each channel captures.
Compared at handler level (interviews out of 4, tickets out of 12):

| Theme | Interviews | Ticket handlers |
| --- | ---: | ---: |
| Callout vanished | 3/4 | 11/12 |
| Gone quiet | 2/4 | 4/12 |
| No live callout signal | **2/4** | **0/12** |
| Text too small | **2/4** | 2/12 |
| Dark mode | 1/4 | 4/12 |
| Filter resets silently | 1/4 | 4/12 |
| Same alert sound | 1/4 | 2/12 |
| Capability tag legend | 1/4 | 2/12 |
| Supply | 1/4 | **10/12** |
| Filter scope problems | **0/4** | **8/12** |
| Account plumbing | **0/4** | **8/12** |
| Workflow requests | **0/4** | **6/12** |
| Duplicate notifications | 0/4 | 3/12 |
| Performance / export | 0/4 | 2/12 |
| Screen reader support | 0/4 | 1/12 |

**Loud in interviews, absent from tickets.** "Tell me when a callout is live"
is 2 of 4 interviews and **zero of 147 tickets** — the largest single gap, and
it is Ambrose's explicit ask for the redesign. Text too small is 2 of 4, both
naming it their one-thing-to-change, against 2 closed tickets. The human cost
saturates the interviews and survives in the tickets only as relayed quotes.

**All over tickets, absent from interviews.** Account plumbing (8 of 12) and
filter scope problems (8 of 12) appear in no interview at all. Workflow
requests: 6 of 12, none. Screen reader support: one handler, filed twice, with
a colleague who cannot use the console — invisible in the interviews.

At 8-of-12 prevalence, a sample of four should have caught something. These
gaps are **channel, not sample**: those topics are ticket-shaped and do not
surface when someone asks "what would you change?" The reverse holds too — a
live callout signal is not a thing anyone files a ticket about.

## The one real disagreement: they name different victims

Both piles found that some responders went quiet. They disagree completely
about who.

| | Named |
| --- | --- |
| Tickets | Farlight, The Undertow, Corporal Ashgrove, Halfmoon |
| Interviews | Vesper, Meteor Mite |
| Overlap | **none** |
| Database | Farlight, The Undertow, Vesper, Meteor Mite |

The intersection is empty. Tickets get two right and two wrong; interviews get
two right and miss two. **Neither pile alone produces the correct list.** The
union gives six, and only the database removes the two false positives. That is
the case for having used all three sources.

## Why the channels differ

Tickets record **events**; interviews record **conditions**.

A phone number that will not save is acute, specific and fixable, so it becomes
a ticket — and three weeks later nobody mentions it, because it was resolved or
worked around. Text that is too small is chronic and livable: no event to
report, no obvious ask, so it never becomes a ticket — but it is the first
thing said when someone asks what to change.

The consequence: **the ticket queue systematically under-weights chronic
experience and over-weights acute plumbing.** Roadmap from tickets alone and we
ship a year of small fixes while Dot still cannot read the screen. Roadmap from
interviews alone and we miss the eight handlers locked out by a slow password
reset, and the colleague using a screen reader.

**Caveats.** The interview sample is four people, so each is 25% — only the
large gaps are defensible. And the interviews were framed as console redesign
research, which suppressed Supply and probably admin topics; some of the
apparent difference is the question Sofia asked.

---

## What the data confirms, and what it can't

**Confirmed from the `pings` table:**

- The threshold at four misses, across all 16 responders.
- The two handlers reporting quiet weeks hold the two responders at four misses.
- No recovery for any of the four within the available window.
- Callouts nobody attended: 5.5% → 11.1%, and not concentrated in the
  suppressed responders' areas.
- Out-of-area pings: 17.1% → 39.4%.
- A second threshold in the ping outcomes: the four suppressed responders miss
  53–64% of the pings they still receive; the other twelve miss 10–18%.
- Three of fifteen areas filed no tickets at all; two of those three hold a
  suppressed responder.
- Sixteen responders across fifteen areas — only Eastgate has two (both Kip's).
  Everywhere else there is one responder and no local bench.

**Not checkable with what we have:**

- What the console actually tells a handler when a ping goes out. The `pings`
  table has no delivery field and no time-to-accept — only who was pinged,
  when, and the outcome.
- Anything after ~6 September. The data ends there; empty later weeks are the
  edge of the dataset, not a further collapse.
- Whether a score reset tool exists. "Never decays, no reset" comes from
  reading the code, not from anyone at Rook confirming it.

**Open questions this raises for the team:**

1. Wen Li — was penalising a miss identically to a refusal intended, and does
   any tool exist to reset a responder's score? (Marcus's 14 Aug question,
   still unanswered.)
2. Sofia / Wen Li — what does the console signal to a handler when a ping goes
   live, and does Dot's chime cover it?
3. Supply — are the four suppressed responders' gear maintenance schedules
   being affected?

---

## What to fix, and in what order

Everything from all three sources, in four tiers. "Evidence" counts handlers
out of 15 — 12 who filed tickets, 4 interviewed, Mr. Ambrose in both.

### Now — this week

| Issue | Evidence | Sources |
| --- | ---: | --- |
| **Restore the ping wait to 90s** (`config.py:9`) | 13/15 | interviews, tickets, data |
| **Un-floor the four suppressed responders** | 4/15 | interviews, tickets, data |
| **Miss ≠ refusal; let the score decay** (`offer.py:30`, `history.py:35`) | nobody reported it | data only |

**1. The ping wait, today.** Thirteen of fifteen handlers have evidence against
it, across all three sources. One line of config. Reversible. The only item
here where the fix is cheaper than the meeting about it, and it takes the
coverage-gap doubling and the out-of-area travel with it.

**2. The four floored responders — first move is a question, not a fix.** The
most severe harm, and the only item getting *worse while we deliberate*: their
miss rate is 53–64%, which looks like people who have stopped watching the
phone. We do not know whether a reset tool exists, so today's action is finding
out.

**3. Miss ≠ refusal, with decay.** Visible only in code and ping data. Without
it the threshold stays armed for whoever has a bad week next.

> **Sequencing trap.** Item 1 alone will pull callout acceptance back up within
> a fortnight. The metric recovers, the tickets stop, and the mandate for items
> 2 and 3 evaporates — while four responders stay silent. **Get all three
> committed before shipping the first.** This is the judgement call most worth
> getting right.

### Next few weeks

| Issue | Evidence | Sources |
| --- | ---: | --- |
| Filter persistence: warn on reset, fix scope | 9/15 | interviews, tickets |
| Live callout signal for handlers | 2/15 | interviews only |
| Instrument `pings`: delivery field, time-to-accept | nobody reported it | data only |

The instrumentation is not optional housekeeping. Without a delivery field we
cannot answer "did the handler get told, and in time" — the question this whole
investigation kept running into.

### This quarter — the redesign Sofia came for

| Issue | Evidence | Sources |
| --- | ---: | --- |
| Text size and badge contrast | 4/15 | interviews, tickets |
| Dark mode | 5/15 | interviews, tickets |
| Account plumbing | 8/15 | tickets only |
| Alert sounds and capability tag legend | 6/15 | interviews, tickets |
| Workflow requests | 6/15 | tickets only |
| Screen reader support | 1/15 | tickets only |

Screen reader support ranks last on volume and should not be last on priority:
one handler cannot use the console at all.

### Not ours — hand over with the evidence

| Issue | Evidence | Sources |
| --- | ---: | --- |
| Requisition queue, failure reports, catalog search | 11/15 | interviews, tickets |

### On the timeout, honestly

We cut 90s to 60s deliberately — "ping timeout tuning" was a committed Q3 item,
so someone wanted dispatch faster. **We cannot price what 90s costs**, because
`pings` has no time-to-accept field. The trade still looks strongly favourable:
a miss burns the full 60 seconds *and* hands the callout to someone further
away, while an accept at 75 seconds ends it — so restoring 90s probably makes
dispatch faster overall, not slower. That is an inference, not a measurement.
Instrument time-to-accept alongside the change rather than claiming it.

### The procedure, revised

Module 1 said: ask Wen Li first, then decide about the timeout. **Split that.**

- **Ask Wen Li today**, and ask a sharper question than Module 1 framed: not
  only "was penalising a miss like a refusal intended" but *is there any way to
  reset a responder's score, and if we restore the 90-second wait, does that
  help the four at all or are they stranded regardless?* This also finally
  answers Marcus's 14 August question, open for eleven weeks.
- **Do not hold the timeout decision for her reply.** She returned to 700
  messages. We have three independent sources and a measured 92s → 62s gap.
  That decision does not need her; items 2 and 3 do.
- **Take all three to Marcus as one proposal**, with her answer if it has
  arrived and as an open question if it has not.

**Lead with this sentence.** Priya was emphatic that this must not become a
conversation about reverting 4.2 — the weight change was long-requested and
reverting it trades one unhappy group for another. Our proposal does not touch
it. On the evidence the weight change is the innocent half; we are reverting
one line of the other half. Say so first, or the room will hear "revert 4.2"
and the objection will arrive before the evidence does.

---

## A note on the grouping

The first pass grouped these by **symptom** — what each handler complained
about, in their own words, with counts. That is the right first cut for
research: "3 of 4 raised this" only means something while the categories are
still in the customers' terms.

This version groups by **cause**, which is what you need in order to decide what
to build. The counts are kept in the matrix at the top so the research evidence
survives the regrouping, but note that once grouped by cause the counts stop
being evidence about users and become evidence about code.

What moved: the original "handler isn't told what's happening" group split in
two. Most of it (Dot, Ambrose) was never a separate problem — it was the
60-second window seen from the handler's side, and belongs in A. Only Kip's
identical alerts stand alone, in B. Ambrose's "I'd rather it warned me" turned
out to be the same complaint as his filter reset, so those sit together in B
rather than in two different buckets.

The original pass also called A3 the "opposite" of A1. It isn't — it's the same
cause further down the chain, and the distinction that matters is that A1 is an
event and A3 is a state you can't leave.

## Two things about the set itself

**These calls were about the console, and the console is section C.** Sofia
booked four interviews to research a redesign. The largest theme by far is a
routing change nobody mentioned by name, volunteered unprompted. Ambrose
apologised for it: "I don't say that to complain about the software generally."

**Dot and Kip have never filed a ticket.** All 147 callout tickets come from
handlers, and these two aren't among them — yet they hold two of the four
suppressed responders. They reached us only because Sofia happened to book an
interview about something else. Nobody joined up their own clues: Dot said "I
hadn't really put those two things next to each other before," and Kip, with
the whole pattern on one screen, said "I just watch the cards."
