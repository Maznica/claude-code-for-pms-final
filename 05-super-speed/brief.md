# A way back

**Brief · Rook Dispatch · for Helen Achebe · 7 November 2026**
Draft 3. Written plainly rather than polished.

---

## The short version

Dispatch keeps a score for every responder based on how often they say yes.
The score decides who gets offered a callout first.

In 4.2 we cut the time a ping waits on a phone from 90 seconds to 60. People
started missing pings simply because they couldn't reach the phone in time.
Our code treats a missed ping exactly like a refusal, so every missed call
took points off their score. A lower score means fewer offers. Fewer offers
means fewer chances to earn points back. Four responders went all the way to
the bottom and stopped getting work.

There is no way back out. Nothing in the code ever raises a score on its own,
and the one line that adds points only runs when a responder receives a ping —
which is the thing they have stopped receiving.

Putting the timer back to 90 seconds stops new people falling in. It does not
get the four out. **The fix is to stop counting silence as a refusal.** That
is one change, it is small, and everything else in this brief follows from it.

This is not only about the four. Their work moved onto whoever was left, and
in Eastgate that is one person — where incidents going unattended have already
doubled.

---

## What happened to Meteor Mite

Meteor Mite covers Eastgate. Kip is their handler — and also The Gale's. Two
coverage cards, side by side on Kip's middle monitor, same city, same week.

| | Before 4.2 | After 4.2 | September |
| --- | --- | --- | --- |
| Meteor Mite | 70 pings | 14 | **1** |
| The Gale | 82 pings | 69 | 19 |

Same area. Same handler. One of them stopped getting work.

Here is the month from Meteor Mite's phone:

| Date | What happened |
| --- | --- |
| 12 Aug | 1 ping, turned down |
| 13 Aug | 2 pings, both missed |
| 14 Aug | 1 ping, missed |
| 15 Aug | 2 pings, both taken |
| 16 Aug | 1 ping, missed |
| 18 Aug | 1 ping, turned down |
| 19 Aug | 1 ping, taken |
| 20 Aug | 1 ping, missed |
| 22, 27, 29 Aug | 1 ping each |
| 5 Sep | 1 ping, missed — and nothing since |

**What Meteor Mite experienced:** the phone went quiet. They texted Kip all
week asking whether something was broken. Kip could see one card dead and one
card busy, with nothing on screen to explain the difference, so Kip said hang
in there. Nobody ever told Meteor Mite what had happened, because there is
nowhere in the product that could have.

**One fact worth holding on to:** Meteor Mite turned down more callouts than
anyone else on the roster — 20 of them, a 28.6% refusal rate — and was still
at the top score on 11 August. Saying no never did this. Not answering did.

---

## Why it kept happening

1. A callout comes in for Eastgate, and Dispatch ranks everyone available.
2. Meteor Mite is ranked first, so their phone rings.
3. It is 11pm and the phone is in a pocket. After 60 seconds the ping gives
   up and the callout is marked **missed**.
4. Our code sends a missed ping through the same function as a refusal. Both
   cost 0.12 points. Saying yes is only worth 0.08.
5. A lower score means a lower rank, which means fewer pings — so the offers
   thin out while the slide is still happening.
6. Fewer pings means fewer chances to earn the 0.08 back, because that credit
   is only paid when a ping actually arrives.
7. Repeat.

The arithmetic is unforgiving. Every silence costs more than a yes earns, so
mixed results only ever travel one way:

> **1.00** on 11 Aug → **0.52** by the 14th → **0.40** by the 20th → **0.00**
> on 5 September.

Meteor Mite took 3 of those 14 pings. It wasn't enough to hold position, and
there was no way to know that from the outside.

At the bottom the trap closes. The score can't go below zero, and from down
there the number of yeses needed just to stay level is lower than it is
higher up — so the four bounce between 0.00 and 0.08 and never climb out.

---

## The cost landed on whoever was left

Suppressing a responder does not reduce the work. It moves it — and it moved
in two different ways depending on how many people covered the area.

### Eastgate: it concentrated on one person

| The Gale | Before 4.2 | After |
| --- | --- | --- |
| Offers per week | 13 | **21** (+62%) |
| Taken per week | 10 | 16 |
| Take rate | 78% | 70% |
| Missed per week | 0–1 | 2–3 |

The Gale is absorbing roughly six extra callouts a week. Their take rate is
falling and they are now missing pings they never used to miss — the 60-second
timer reaches them too, they are simply high enough up the range to absorb it.
That is the "Gale's exhausted" Kip described, in numbers.

**But the serious part is what it did to coverage.** The share of Eastgate
callouts offered to The Gale *and nobody else* went from **39% to 70%**. When
Meteor Mite dropped out, Eastgate quietly became a one-responder area.

Eastgate callouts are offered to an average of 1.35 people — before and after,
unchanged. So when the one name says no, there is usually no second name.
"Nobody came" in Eastgate **doubled, 5.1% to 10.5%**, and of the six failures
since 4.2, **four are The Gale alone**:

| Date | Incident | What happened |
| --- | --- | --- |
| 20 Aug | A chess grandmaster froze the fountain in the square | The Gale — turned down |
| 24 Aug | A sit-in at the power station | The Gale — missed |
| 25 Aug | A clown stealing streetlight bulbs | The Gale — turned down |
| 31 Aug | A marching band jamming every radio in the market | The Gale — turned down |

Nobody attended. Not because nobody would — because there was nobody left to
ask.

This reframes the fix. For Meteor Mite it is being let back in. For The Gale it
is **getting the right to say no again**: today a refusal from The Gale means
an incident goes unattended, which is an unreasonable weight to put on one
person without ever telling them they are carrying it. And for Rook it is
coverage, not fairness — four incidents in three weeks is a service failure,
not a morale problem.

### Everywhere else: it was exported to other areas

Eastgate had only two responders, so the work had one place to go. The other
suppressed areas had neighbours, and the work crossed the city instead.

**Old Town** (Vesper's area):

| | Vesper | Responders from elsewhere |
| --- | --- | --- |
| Before 4.2 | 72 pings | 10 |
| After | **10** | **52** — Nightwell 21, Captain Vantage 19, The Longcast 12 |

**Uptown** (Farlight's area) is the same shape: Farlight went from 59 pings to
6, while Sgt. Falkirk, Cindermark and Sgt. Bulwark took 45 between them.

So responders are now driving into other people's areas to cover for someone
who is available, at home, and not being asked. Nightwell — the busiest
responder on the roster — went from **3 out-of-area callouts a week to 12 of
21**. Over half their work is now outside Midtown.

**None of this is visible anywhere.** A ping taken forty minutes away counts
in acceptance rate exactly the same as one next door, so the headline metric
is blind to it. Nightwell's console card reads *Normal* and shows the busiest
numbers at Rook. Nothing on it says that half that work belongs to somebody
else's part of the city.

This is the roster-wide out-of-area shift — 17% to 39% — with names attached.
It is also the cheapest thing on this list to measure: we cannot calculate
travel time, but we already record which area a callout was in and which area
a responder covers. Comparing those two columns is a query, not a project.

---

## Wen's question, and the answer

Wen Li left a note beside this code in 2019:

> *should this ease back toward NEUTRAL_SCORE on its own after a while? […]
> Against: if somebody has stopped taking work, we probably want that to
> stick until they take work again.*

She was right to ask, and the "against" case is the part that doesn't hold
up. It assumes a responder can take work again. They can't. The score decides
whether a ping arrives, and points are only added when a ping arrives. "Until
they take work again" is a door that locks from the inside.

Seven years on, the data answers her: it should ease back, because the
alternative is permanent.

**And this closes Marcus's question from 14 August.** He asked whether the
4.2 change was meant to apply to responders who had been turning jobs down.
It wasn't, and it couldn't have been — there is no line anywhere in the
routing code that looks at who a responder is. Nobody was singled out and
nobody was grandfathered in.

---

## What we'd build

**1. A missed ping is not a refusal.**
Split the two apart. Turning a job down still costs the full 0.12 — that is a
decision somebody made. Missing a ping costs much less, or nothing. Our own
glossary already says these are two different things. The code just doesn't
honour it. This is the whole root cause, and it is a small change.

**2. A way back.**
Two parts, and it needs both:

- *Scores ease back toward the middle over time*, so a bad fortnight expires
  instead of becoming a life sentence. The direct answer to Wen's note.
- *A floor of offers:* anyone available gets reached a set number of times a
  week whatever their score. Easing back is too slow on its own to rescue
  somebody already at the bottom.

**3. Both people can see it happening.**

*On Kip's console card:*
> Meteor Mite is ranking low on recent acceptance: 3 pings missed 13–18 Aug.
> Expected back to normal by 2 Sept.

Kip told us they don't even know what they'd want the screen to say — only
that it shouldn't look like a coincidence at midnight. That sentence is what
stops it looking like one.

*On Meteor Mite's phone, on day two rather than never:*
> You've missed 3 of your last 5 pings. That's moving you down the order.
> Take the next one and it comes back.

Plus a single tap for **not available tonight**, so somebody who genuinely
can't answer can say so instead of being counted as silent.

### Un-sticking the four: what number, and what floor?

First, where everyone actually stands. Replaying every ping since June:

| Responder | Score today |
| --- | --- |
| Ashgrove, Bulwark, Halfmoon | 1.00 |
| Longcast, The Gale, Cindermark | 0.96 |
| Captain Vantage | 0.92 |
| Nightwell | 0.88 |
| The Drift | 0.84 |
| Falkirk, Stormwrack | 0.76 |
| Ironvale | 0.72 |
| **Meteor Mite, Vesper, Farlight, The Undertow** | **0.00** |

Twelve people between 0.72 and 1.00. Four people at exactly zero. Nothing in
between — this is a cliff, not a distribution.

**There is no threshold to clear.** Nothing in the code says "below X you
stop getting work." Ranking is relative: you get the callout if you beat
whoever else is free that night. So the only question that matters is where a
number puts somebody *relative to the twelve*.

That is what rules out 0.5. It is the tidy answer — `NEUTRAL_SCORE`, the
documented default we give someone we know nothing about — but it would leave
all four **below every single healthy responder on the roster**, two notches
under Ironvale at 0.72. We'd have moved them from last to last.

**0.8 is the right number, for two reasons that arrived independently.**
It is where the counterfactual replay puts them — run August again without
penalising misses and the four land at 0.76–0.84. And it is mid-pack today:
above Falkirk, Stormwrack and Ironvale, just under The Drift. In Eastgate it
puts Meteor Mite at 0.80 against The Gale's 0.96, a gap worth under a minute
of equivalent travel time, so proximity decides who goes — which is exactly
what should decide it.

This is restitution, not a favour. We are putting them where they would have
been if we had not broken it.

**Why the floor still has a job.** Not as the recovery mechanism — as a
backstop and a detector. If we restore the score and the work *still* doesn't
come, the problem was never ranking, and we need to know that. Given we
average 1.29 pings per callout, a short candidate list is a live suspicion.

So the floor can be small: **two or three offers a week, not five** — enough
to catch the failure case without putting the four in a side lane. Sizing
against the real volume:

| | |
| --- | --- |
| Callouts per week | ~132 (range 110–144) |
| Responders on the roster | 16 |
| Offers per responder in a normal week | **~10** |
| Pings per callout | 1.29 |

One a week would have been a token — a tenth of a normal week, and months to
climb anywhere from zero. Five would have been the whole recovery, which is
the wrong shape once the score is restored properly.

**Two constraints.** We can only offer callouts to someone marked available —
`available_for()` is the only real filter and routing never changes it, so
the floor works inside availability and cannot conjure work for someone who
isn't free. And decay alone can't do this job: easing back goes *toward*
0.5, so it would never lift anyone to 0.8. Decay prevents the next case; it
does not fix this one.

**Order matters here.** Setting the score before fix 1 ships would be
pointless — the four currently miss 53–64% of what they get, and under
today's code every one of those misses costs 0.12 again. Fix 1 first, then
the numbers hold.

### What we're not doing

Reverting the proximity change from 4.2. Responders covering wide areas asked
for it for three quarters, the arithmetic says it makes this spiral *less*
severe rather than causing it, and undoing it just swaps one unhappy group
for another. Priya's position, and still the right one.

---

## The sequence, side by side

| Step | Today | With the fix |
| --- | --- | --- |
| Callout arrives | Dispatch ranks everyone available | Same |
| Ping sent | Waits 60 seconds | Timer decided separately, on evidence |
| No answer | Marked missed | Marked missed |
| Score | −0.12, same as a refusal | No penalty. A refusal still costs 0.12 |
| Responder learns | Nothing, ever | Phone tells them while they can still act |
| Can't take work tonight | Silence, which costs points | One tap: *not available tonight* |
| Over time | Score sticks where it fell | Eases back toward the middle |
| At the bottom | No offers, so no way to recover | Score restored to 0.8, plus 2–3 offers a week as a backstop |
| Handler sees | Two cards, no explanation | A line on the card saying why it's quiet |
| We see | Acceptance rate recovers; four people don't | Offers per responder, so the gap is visible |

---

## What we already know, and don't need to ask

Both questions I had queued for Wen Li are answerable from the source. They
are worth her review, but they are not blockers.

**Does Dispatch ever offer a callout to more than one person?** Yes — the
dispatch loop walks the whole ranked list until somebody says yes. So when a
callout dies after a single ping, which it did in 82 of 97 failures, the list
itself had one name on it. Across all callouts we average **1.29 pings each**.
That reframes things: short cascades are an *availability* problem, not a
ranking one. Ranking decides the order of a list that is usually very short.

**Is there a way to reset a responder's score?** Not in the routing service —
there is no reset, no decay and no timestamp anywhere in it. The scores live
in a plain in-memory dictionary. Either a restart wipes everyone back to the
0.5 default, or persistence lives somewhere outside these files. That is the
one thing genuinely worth a message to Wen, and it changes *how* we un-stick
the four, not *whether* we do.

---

## How we'd know it worked

Not by acceptance rate. That number recovers on its own the moment the timer
goes back to 90 seconds, and it is the number that hid this for seven weeks.

- **Nobody is stuck.** How many responders sit at or near the bottom for more
  than a week. Target: zero. Replaying August with missed pings unpenalised,
  none of the 16 reached the bottom. That is the shape we want.
- **The spread, not the average.** Offers per responder per week against the
  ~10 baseline. Eastgate's −59% / +74% split is the thing that must not
  happen again, and nobody at Rook reports this today.
- **The four by name.** Meteor Mite, Vesper, Farlight and The Undertow back in
  rotation and taking work — or we know why not.
- **Coverage, per area.** "Nobody came" by area, and the share of callouts
  offered to one responder only. Eastgate is at 70% on that second measure and
  10.5% on the first. Both should fall. Neither is reported today.
- **Out-of-area share, per responder.** Nightwell is at 12 of 21 against a
  baseline of 3 a week. It is a two-column comparison we can already run, and
  it is the only measure here that catches a responder being worn down by
  travel rather than by volume.
- **Kip stops guessing.** A handler can answer "why is it so quiet" from the
  console instead of texting a responder to hang in there.

---

## The order I'd do this in

1. **Stop counting a miss as a refusal.** The root cause, the smallest
   change, and the only one that stops this happening to anybody else. Ship
   it first and on its own.
2. **Un-stick the four.** Set them to 0.8 — mid-pack, and where the
   counterfactual says they'd have been — by whatever mechanism exists once
   Wen confirms where the scores live. Add a floor of two or three offers a
   week as a backstop, not as the rescue. In the last fortnight of data we
   have, Meteor Mite, Vesper,
   Farlight and The Undertow were offered 1, 2, 3 and 3 callouts and took
   **none**. That data is two months old. This gets harder every week it
   waits, because at some point it stops being a scoring problem and becomes
   four people who have stopped expecting the phone to ring.
3. **Record the ring.** Add delivery and time-to-accept to the ping record.
   Today "missed" covers both *the phone never rang* and *it rang and nobody
   moved*, and we cannot tell them apart. Without this we can't judge the
   timer, and we can't honestly tell a responder they missed something we
   can't prove reached them.
4. **Check the gear.** Supply schedules maintenance off the availability
   record Dispatch writes. Four responders have now had two artificially
   quiet months. Has their servicing drifted with it? Nobody has looked, and
   it isn't answerable from the Dispatch side.
5. **Tell the handler what's going on.** The console line, so Kip has a real
   answer. The responder-facing message follows once step 3 makes it honest.
6. **Then decide the timer.** 90 seconds, 60, or something else — a decision
   with Marcus and Helen, taken with step 3's measurements in hand rather
   than as a reflex.

**Why the timer goes last, when everyone expects it first.**

Because it is the one change that makes the headline acceptance rate
recover — and once that number is healthy, the attention goes with it.
Steps 2 to 5 are all harder to fund the week after the graph looks fine.

And because the inflow appears to have stopped. All four fell in during a
single five-day window in mid-August. In the last fortnight of data the other
twelve responders were each offered 14 to 41 callouts, taking 10 to 30 of
them, with miss counts of 1 to 4. Nobody was mid-slide. There is no gradient
here — there are twelve people who are fine and four who are gone.

**The caveat, and it is a real one.** Our ping data stops on 6 September and
it is now November. Everything above describes a situation two months stale,
and nobody has refreshed it. Step 1 should travel with a request to Ravi
Menon for current numbers — because if anyone *has* fallen in since, the
timer moves sharply up this list.

---

## What's still missing

1. **How long is "back to normal"?** Both messages above promise a recovery
   time and we haven't picked an ease-back rate.
2. **Security Policy 4.1 is unread.** Showing a responder their own standing,
   and showing a handler why someone ranks low, is new disclosure about a
   person. It has to clear policy before design, not after.
3. **No design, no estimate.** Sofia Marino hasn't seen any of this, and it
   lands on the same console card her redesign research covers — where Kip's
   actual top request is dark mode.
4. **We have no way to reach responders.** All 147 tickets came from handlers.
   Kip and Aunt Dot, who between them handle two of the four worst-hit, filed
   none. A message on the phone app is a direction we have never used, and we
   have no way to hear back.
5. **What does this push out?** It isn't on the Q3 roadmap. 4.3 currently
   holds Requisition approval chains. And the conversation about what got
   squeezed out of 4.2 — Availability Confidence, committed and missing from
   the release notes — still hasn't happened. **Helen, same conversation.**
6. **The availability finding needs following.** If most callouts only ever
   have one candidate, that is a bigger story than this brief, and it is the
   next thing I'd look at after these six steps.
