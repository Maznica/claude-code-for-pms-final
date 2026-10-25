# Rook Dispatch — overview of my first questions

New PM on Rook Dispatch. Written after reading our own sources, in plain
language. Nothing here is invented — every point traces back to a file, a
wiki page, or a query against our database.

**Sources used**

- Wiki: About Rook, Glossary, Team directory, Releases, Q3 roadmap, the four
  product briefs, the Release 4.2 page and its comments, the September
  customer interviews
- Repo: `00-rook/code/dispatch-routing/` (the routing code),
  `00-rook/company/notes/handoff-from-priya.docx`
- Database: `pings`, `callouts`, `responders`, `handlers`, `support_tickets`
  (29 June – 6/7 September 2026)

**One limit worth stating up front:** our data begins on 29 June 2026. Any
claim about "last year" or "every August" cannot be checked with anything
Rook holds.

---

## The product, in plain words

### What Rook sells

Rook Industries makes software for superheroes — and, more importantly, for
the ordinary people who look after them.

| Product | What it does | Ours? |
| --- | --- | --- |
| **Rook Dispatch** | Gets the right hero to the right emergency, fast | **Yes** |
| Rook Supply | Keeps their gear working and accounted for | No |

We do not employ any responders. They operate independently and buy our
software; we charge a subscription per active responder. About Rook notes
that of 241 staff, **240 have never met a responder**.

### The problem we solve

Before software this was a person with a whiteboard and a phone list: work
out fast who can reach the incident, who is free, and who can actually do
this kind of job — then ring them one by one until someone says yes.

Dispatch does that thinking in a second. Our one-pager: get the right
responder to the right incident *"faster than a human coordinator working
from a whiteboard and a phone list."*

### How it works

1. Something bad happens.
2. A handler types it into the console.
3. Rook ranks everyone available — closest, right skills, recent yeses.
4. One phone buzzes. **60 seconds** to answer.
5. Yes → the hero goes. No, or no answer → the next hero on the list.
6. List runs out → **nobody came**.

"Nobody came" was **5.5%** of jobs before 12 August and is **11.1%** since.
It doubled.

### Our size (database, 29 June – 6 September 2026)

16 responders · 15 handlers · 15 areas of the city · 1,319 callouts
(about 19 a day) · 147 support tickets

A small, tight world. Every affected person has a name.

### Who our customers are

| Who | Where | What they do |
| --- | --- | --- |
| **Responder** | A phone in their pocket | Wears the mask. Gets the buzz, says yes or no, goes |
| **Handler** | A web console | Looks after one or two responders. Enters incidents, watches coverage, sets availability |
| *Quartermaster* | A web console | Owns the equipment. **Supply's user, not ours** |

The handler is who is actually clicking. The responder has a phone and a
thumb; the handler has the screen and the relationship.

They are not call-centre staff. From our September interviews: **Aunt Dot**
works the console *"on the kitchen counter next to the toaster,"* standing
up, while making dinner. **Kip** works nights with three monitors and has
asked for dark mode every time we speak to him.

One rule shapes everything: **we never store who the responders really are.**
We hold capability tags and availability, never a legal identity. About Rook:
*"Do not attempt to work out who anyone is. Two people have been let go for
this."*

### Handler and responder — how they actually relate

The common misreading is that Rook calls the hero and the hero says yes or
no. That part is exactly right. **The handler is never in that moment.** The
handler does everything around it.

Think of a football coach. The player takes the shot; the coach never does.
But the coach picks the team sheet, says who is injured, and talks to the
referee when something is wrong.

Glossary: the handler is *"the person responsible for a specific responder or
small group of them — their availability, their gear, their readiness. The
handler is usually who is actually clicking around in our product."*

**How a callout actually runs**

1. **Before anything happens** — the handler sets him up. Our one-pager: the
   handler *"manages a responder's availability windows and capability
   tags."* In plain words: *free Tuesday night*, *can fly*, *good in cold
   weather*. The responder can also set his own availability from the phone;
   both can.
2. **The emergency arrives** — typed in by a handler, or pushed from an
   intake system.
3. **One buzz goes out — to the responder's phone and nowhere else.** The
   handler cannot accept for him. There is no handler app; it is still only a
   Q4 proposal in Product briefs. The handler gets no alert at all.
4. **Afterwards** — the handler is the one who tells us something went wrong.

Step 3 is why Aunt Dot's interview reads the way it does: she hears the buzz
on the counter, shouts upstairs, he comes down two steps at a time, and the
job is gone before his thumb lands. Her one request was *"maybe something
that tells me too, not just him."*

**Who can do what**

| Only the handler | Only the responder |
| --- | --- |
| Type in an emergency | **Answer the buzz** |
| Watch the coverage board | Say yes or no |
| **Override Rook's choice** and pick someone else | Set his own availability |
| Set availability windows and capability tags | |
| Order gear and report it broken (Supply) | |
| Tell Rook something is wrong | |

Overriding is a real power — a handler can overrule the ranking entirely. We
only began recording when they do in Release 4.0.

**"Handler" means very different things to different people**

- **Aunt Dot** looks after Vesper from her kitchen counter and cooks his
  dinner — *"he's a good boy, he just doesn't eat enough."* Family.
- **Mr. Ambrose** calls Captain Vantage *"the young master"* and works from a
  study with a second monitor. Staff.
- **Kip** runs two responders on a night shift with three monitors and a
  headset. A professional operator.
- **Halloran** also runs the gear cage and checks in twice a shift —
  *"Bulwark's a pretty steady responder, doesn't need a lot of babysitting."*

15 handlers for 16 responders. Kip covers two (Meteor Mite and The Gale).

**Why this changes how I read the problem**

About Rook: *"When a responder is unhappy with the product we hear about it
from their handler, and on two occasions through a wall."*

Responders never contact us. All 147 support tickets were filed by a handler,
about a responder. **The handler is our only ear** — so our picture of what is
wrong is only as good as which handlers choose to speak.

Aunt Dot and Kip have filed zero tickets, and their responders — Vesper and
Meteor Mite — are two of the four cut out of the work. Aunt Dot assumed the
silence was normal and apologised for raising it. Kip told his responder
*"hang in there."*

The two people hurt worst are precisely the two we would never have heard
from. We only know because Sofia rang them about a screen redesign.

That is not a data problem. It is a design problem: **our only feedback
channel is a person who may be too kind, too busy or too polite to use it.**

### What hurts right now

**Responders**

- *Gone before I could answer* — sixty seconds is not enough
- *My phone never goes off* — weeks with no jobs at all
- *I think I have been forgotten* — nobody can tell them why

**Handlers**

- *I hear it before he does* — Aunt Dot, "a buzz through a ceiling"
- *Two heroes, two stories* — Kip, one dead quiet and one on fire, same week,
  same city
- *I have nothing to tell them* — Kip, "hang in there, it'll pick up —
  because what else do you say"

The handler's real pain is not the buttons. It is having **no explanation**.
We built the machine that made the decision and never gave them a way to see
why.

### Who's who on Dispatch

| Role | Name | Owns |
| --- | --- | --- |
| Director of Product | Helen Achebe | Roadmap and commitments |
| **Product Manager** | **me** | Took over from Priya, who left 21 August |
| Engineering Manager | Marcus Oyelaran | The engineering team |
| Staff Engineer | Wen Li | Built the part that picks who gets called |
| Product Designer | Sofia Marino | Console and phone app; ran the interviews |
| Data Analyst | Ravi Menon | The numbers, weekly |
| Support Lead | Nadia Hoffmann | The tickets |

Priya's handover on Wen Li: *"If you want to actually understand how a
responder gets ranked, it's a conversation with her, not a document — there
isn't a good document."*

### How we measure it

| Measure | Plain words | Where it stands |
| --- | --- | --- |
| **Acceptance rate** | Of every 100 buzzes, how many get a yes | 77 before August, 64 after |
| Time-to-accept | How fast they say yes | Not stored in our database |
| Coverage gap | Nobody had the right skills | Counted apart from "nobody would" |

Acceptance rate is the headline. Priya: *"get comfortable explaining it
early."*

### In one paragraph

Rook Dispatch replaces a person with a whiteboard. When an emergency arrives
it ranks every available responder and offers the job to one phone at a time
until someone accepts. Our customers are the responders and — mostly — the
handlers who look after them, often from a kitchen counter at night. We are
fixing it because since 12 August the machine has been quietly cutting four
of our sixteen responders out of the work, and the people it hurt most are
the ones who never complained.

---

## Question 1 — What's contradictory or missing?

Information about Dispatch lives in three places, and they disagree.

| Place | What it is | In plain words |
| --- | --- | --- |
| The wiki | Pages people wrote | What we *say* we do |
| The database | A record of every call | What *actually happened* |
| The code files | The machine's real rules | What the machine *really does* |

### 1. The wiki says four things decide who gets called. The code uses three.

The Glossary lists proximity, availability, capability and recent acceptance
as inputs to routing priority. `routing.py:38-42` scores only three —
availability is a filter applied before scoring, not a scored input. You
can't tune a dial that doesn't exist.

The Glossary was last updated **4 August 2026**, eight days before Release
4.2. Nobody revised it afterwards.

### 2. Our dictionary says "missed" and "turned down" are different. The machine treats them identically.

The Glossary defines *missed* as "recorded separately from turned down."
`offer.py:30` sends both down the same path, and `history.py:35-39` states it
outright: *"Same either way."*

A responder in the shower is penalised exactly as hard as one who read the
job and refused it.

**Marcus asked about this on 14 August** — he noted "the config doesn't
distinguish between them." He was right, and the question is still
unanswered. Wen Li, the only person who knows, was away 14–24 August and
returned to 700 messages.

### 3. The code's README says everyone gets asked. In practice four responders stopped being asked at all.

`README.md:12-13` — *"Everyone available is on the list."* True, but
`offer.py:25-31` stops at the first yes. Rank low enough and the job is
always taken before it reaches you.

Calls received 1–6 September 2026 (the same six days for everyone):

| Responder | Calls | Calls before 4.2 (29 Jun – 11 Aug) |
| --- | --- | --- |
| Nightwell | 20 | 95 |
| The Gale | 19 | 82 |
| Stormwrack | 18 | 82 |
| Captain Vantage | 17 | 76 |
| *(nine others)* | 7–12 | — |
| Meteor Mite | 1 | 70 |
| The Undertow | 1 | 75 |
| Vesper | 1 | 86 |
| Farlight | **0** | 75 |

### 4. "It's seasonal" is unverifiable, and what we can see argues against it.

Priya (19 August, and in her handover): *"August is always soft for us, every
year."* Marcus repeated it on 26 August. Both have treated it as settled.

Our database contains no previous August. The claim cannot be tested.

What is testable: the first eleven days of August were normal (77.9%
acceptance in the week of 3 August), then acceptance dropped to 54.2% in the
week containing **12 August**, the day 4.2 shipped. Daily, it goes 68.4% on
the 11th to 44.0% on the 12th. Seasons do not begin on a Wednesday.

### 5. The support tickets systematically under-report the worst cases.

| Responder | Calls 1–6 Sept | Tickets filed by their handler since 12 Aug |
| --- | --- | --- |
| The Undertow | 1 | 17 (Desmond Okafor) |
| Farlight | 0 | 16 (Linda Pruitt) |
| Vesper | 1 | **0** (Aunt Dot) |
| Meteor Mite | 1 | **0** (Kip) |

Two of the four worst-affected responders have handlers who filed nothing.
We only know about them because Sofia interviewed them in September for an
unrelated console redesign:

- **Aunt Dot** (3 Sep): *"there have been stretches, more than one, where the
  phone just sits there and sits there."* She assumed it was normal.
- **Kip** (4 Sep), who handles two responders in the same area: *"Meteor
  Mite's been texting me all week asking if something's broken because it's
  so quiet. And The Gale hasn't stopped, like, at all, same week, same
  city."* He read it as a console display problem.

The ticket pile describes who writes tickets, not who was harmed.

### 6. Four things nobody wrote down anywhere

- **No description of how routing works.** Priya's handover: *"There is no
  written description of how we decide who gets pinged. I kept meaning to
  write it. Please write it."* The knowledge sits with one engineer who was
  away for the fortnight the problem emerged.
- **No reason recorded for the 4.2 settings.** `config.py` records that
  proximity went 0.45 → 0.60 and recent acceptance 0.40 → 0.25. Nothing
  anywhere records why those values, who chose them, or what was modelled.
  The file warns "don't change anything in here without telling the
  engineering manager" — and he is the one asking what it was meant to do.
- **`00-rook/feedback/` is empty.** Only a `.gitkeep`.
- **No delivery record on pings.** The `pings` table has no "sent",
  "delivered" or "opened" column. A notification that never arrived and one
  that arrived with 60 seconds on the clock are recorded identically as
  `missed`. This is why Nadia could not explain two thirds of her tickets —
  the information does not exist in any Rook system.

### 7. Roadmap and release notes don't reconcile

- **Availability Confidence** is listed on the Q3 roadmap as Committed to
  4.2. It is absent from the 4.2 release notes, and not marked deferred. The
  roadmap still reads "Committed", last reviewed 30 June, owner Priya — who
  left on 21 August. The Releases page states: *"Items deferred from a
  release are not listed here."*
- All six roadmap items still list Priya as owner. The Team directory was
  updated 2 September; the roadmap was not.

### 8. Two "proposed" briefs describe work that already shipped

- **Bulk Callout** (12 Jan) — *"nobody's picked it up yet."* Shipped in 4.1,
  16 June. Its own note asked for a baseline "before this ships"; it shipped
  without one.
- **Routing Override Audit Log** (5 Mar) — *"ready to build,"* waiting on the
  profile redesign. Both shipped in 4.0, 7 April.

### 9. A brief that contradicts itself in one paragraph

**Requisition Approval Chains** states the ask, writes *"That's the whole
ask,"* then adds four more features in the next sentence. It is committed to
4.3 with scoping listed as TBD. Meanwhile Halloran's September interview
describes an eleven-day wait on a cracked vest plate — a queue
prioritisation problem the brief does not address.

### 10. Organisational gaps

- **Ravi Menon** is our Data Analyst, who "reports weekly on how often
  responders are taking their pings." He appears nowhere in the 4.2
  discussion. On 18 August Nadia asked "can someone pull the trend" and
  Marcus replied "I can get you something rough but it won't be the real
  weekly numbers." Priya's handover routes the incoming PM to the engineering
  manager for data rather than to the analyst.
- The weekly acceptance report Ravi owns is never referenced. Either it
  exists and nobody read it, or it doesn't and the directory is wrong.
- **Helen Achebe** owns "roadmap and commitments" and appears in no thread
  about a release that missed a commitment.
- Four handlers (Aunt Dot, Kip, Halloran, Mr. Ambrose) are recorded by
  nickname where the other eleven use full names. Worth checking against
  Security Policy 4.1 before designing anything touching responder records.

---

## Question 2 — How routing works, and what 4.2 changed

**The flow.** Filter to whoever is available, score them, sort, then offer
the callout to one responder at a time until someone accepts
(`offer.py:19-31`).

**The score** (`routing.py:38-42`):

| Component | Before 4.2 | After 4.2 |
| --- | --- | --- |
| Proximity | 0.45 | **0.60** |
| Recent acceptance | 0.40 | **0.25** |
| Capability match | 0.15 | 0.15 |

- Proximity = `1 − (travel_minutes / 45)`, zero at or beyond 45 minutes
- Capability = fraction of required tags held
- Recent acceptance = running score in [0, 1], starts at 0.5,
  **+0.08** per accept, **−0.12** per turn-down *or miss*, **never decays**
  (the 2019 TODO in `history.py:30-34`)

**What 4.2 changed:** three constants in `config.py`. No logic, no new code.

```python
OFFER_TIMEOUT_SECONDS = 60           # was 90 until 4.2
WEIGHT_PROXIMITY = 0.60              # was 0.45 until 4.2
WEIGHT_RECENT_ACCEPTANCE = 0.25      # was 0.40 until 4.2
```

**The interaction nobody modelled.** The weight change alone would have
*reduced* any ranking spiral. The timeout cut multiplied how often the
penalty fires — missed pings went from 2.3% to 18.0% of all pings. Each
penalty is weaker; they arrive roughly eight times as often, never decay, and
the penalty removes the only thing that reverses it, because you can only
earn credit by being pinged.

From the starting 0.5, four misses puts a responder at 0.02 — effectively
floored. At 60 seconds, that is about a week.

| Responder | Missed in first 5 days after 4.2 | Calls 1–6 Sept |
| --- | --- | --- |
| The Undertow | 5 | 1 |
| Meteor Mite | 4 | 1 |
| Vesper | 4 | 1 |
| Farlight | 4 | 0 |
| Stormwrack | 3 | 18 |
| The Gale | 2 | 19 |
| Nightwell | 2 | 20 |

Everyone who missed four or more is gone. Everyone who missed three or fewer
is fine. No exceptions in either direction.

---

## Question 3 — Ignoring, or not reaching?

**Neither.** Responders are declining *less* than before.

| Outcome | Before 4.2 | After 4.2 | Change |
| --- | --- | --- | --- |
| Taken | 76.6% | 64.0% | −12.6 pts |
| Turned down | 21.1% | 18.0% | −3.1 pts |
| **Missed** | **2.3%** | **18.0%** | **+15.7 pts** |

−12.6 = −15.7 + 3.1. The entire decline is missed pings. Refusals improved.

**"Gone before I could answer"** is the timeout cut. Handlers describe the
phone *arriving* — Mr. Ambrose (ticket 3043), Owen Bramwell (3048), Teresa
Alvarez (3052), and Aunt Dot: *"It didn't used to feel like a fair race,
phone to stairs, and now it does, and he loses it more than he wins it."*

**"Phone never goes off"** is not a delivery failure — **no ping was sent.**
There are no rows in the database to deliver. Vesper was pinged first on 75
callouts before 4.2 and on 11 after; The Undertow 62 → 12; Farlight 60 → 8;
Meteor Mite 50 → 9.

Theme 1 is theme 2, one week later, for whoever was unlucky in the first five
days.

**Can delivery failure be ruled out?** For theme 1, conclusively — nothing
was sent. For theme 2, the evidence is strong but not provable from our data,
because we don't record delivery (see Q1, item 6). Against a delivery fault:
misses are spread evenly across all 24 hours with no outage shape; the step
change lands on 12 August and no other date; 4.2 shipped no code to the push
path; and 4.1 (which *did* touch push reliability, 16 June) was followed by a
2–3% miss rate.

**On the recovery.** Aggregate acceptance is climbing back (54.2% → 72.7% by
31 August). That will look like the seasonal read being vindicated. It isn't
— the surviving responders are being pinged more and accepting, while the
four suppressed ones are excluded from the denominator. Their absence makes
the number look better.

---

## Hypothesis register — 15 things that might have gone wrong

My own analysis, ranked by how sure the evidence makes me. Includes the
explanations I think are **wrong**, because some of them are currently
driving decisions at Rook.

**The chain I believe actually happened**

1. We shortened the answer time: 90 seconds → 60.
2. Far more calls got missed: 2 in 100 → 18 in 100.
3. A miss counts as a refusal — same punishment either way.
4. His score drops and never heals.
5. He sinks to the bottom; the job is taken before it reaches him.
6. No calls means no way back — you only earn points by answering one.
7. He vanishes from the work: 0 or 1 call in six days.

Step 6 loops back to step 5. Being punished is what stops the calls, and
answering calls is the only way to undo the punishment. There is no exit from
inside.

### Group A — Inside the machine

**1. We shortened the answer time from 90 to 60 seconds** — *almost certain*
`config.py:9`: `OFFER_TIMEOUT_SECONDS = 60  # was 90 until 4.2`. Missed calls
went from 2 in 100 to 18 in 100 overnight. Mr. Ambrose's responder was "coat
on, one boot on" when the job moved along.

**2. A missed call is punished exactly like a refusal** — *almost certain*
`offer.py:30` sends both down the same path; `history.py:35-39` says "Same
either way." Our Glossary says these are two different things. The machine
disagrees with our own dictionary. This is what turns a short timer into a
punishment machine.

**3. The score never heals on its own** — *almost certain*
`history.py:30` carries a note marked **TODO(2019)** asking whether the score
should drift back toward neutral, answered with "Leaving it as-is for now."
That "for now" is seven years old, and it is why a bad week in August is
still a bad week in October.

**4. Getting fewer calls removes the only way to recover** — *almost certain*
+0.08 for a yes, −0.12 for a miss. You can only say yes to a call you
receive, and the punishment is what stops calls arriving. Everyone who missed
four or more in the first five days is gone; everyone who missed three or
fewer is fine. No exceptions in either direction.

**5. Two changes shipped together and nobody checked the combination** —
*confirmed fact*
The Q3 roadmap lists "Change to who gets pinged" and "Ping timeout tuning" as
two separate items. They shipped the same morning. Nothing records that
anyone asked what happens when you combine them. Neither change alone does
this damage.

**6. The new ranking weights, on their own** — *possible, probably not the
main cause*
This is where I disagree with the house view. The company talks about 4.2 as
"the change to who gets pinged," but we *lowered* how much acceptance history
counts (0.40 → 0.25), which makes a bad score matter **less**. In proximity
terms, a floored responder used to forfeit ~40 minutes of travel advantage;
now ~19. The weight change should have softened this, not caused it. I cannot
fully separate the two changes, so I will not rule it out — but we may be
blaming the wrong half of the release.

### Group B — Explanations that don't survive the evidence

**7. "It's just August"** — *unlikely, and untestable*
The house view since 19 August. Our database starts 29 June 2026; there is no
previous August in it. What we can see argues against it: 11 August was 68%
acceptance, 12 August was 44%. Seasons do not start on a Wednesday afternoon.

**8. The buzzes aren't arriving** — *unlikely, but unprovable*
Missed calls are spread evenly across all 24 hours; real delivery faults
cluster. 4.2 changed no notification code. Handlers describe the phone
buzzing. But `pings` has no delivery column, so a buzz that never arrived and
one that arrived with 60 seconds on the clock are identical in our data.
Unlikely is not "no."

**9. The heroes are simply refusing more** — *ruled out*
Refusals went **down**, 21 in 100 → 18 in 100. Every bit of the decline is
missed calls. Nobody got lazy. If anyone proposes a scoreboard or a nudge
campaign, this is the number that stops it.

### Group C — Why nobody caught it for ten weeks

**10. The worst-hit handlers never wrote to us** — *very likely*
Aunt Dot and Kip filed zero tickets; their responders, Vesper and Meteor
Mite, are two of the four cut out. All 147 tickets came from handlers. The
ticket pile shows who writes tickets, not who got hurt.

**11. We never asked the person whose job is the numbers** — *confirmed fact*
Ravi Menon's listed job is the weekly acceptance report. On 18 August Nadia
asked "can someone pull the trend?" and Marcus answered "I can get you
something rough but it won't be the real weekly numbers." Ravi appears
nowhere in the thread.

**12. We never record whether the phone actually rang** — *confirmed fact*
This is why Nadia could not explain two thirds of her tickets. Not because
she missed something — because the information does not exist in Rook.

**13. Both people who understood it were gone at once** — *confirmed fact*
Wen Li built the ranking and was away 14–24 August, the exact fortnight it
broke. Marcus asked the right question on the 14th; she replied "away until
the 24th" and returned to 700 messages. Priya left on the 21st with no
handover overlap.

### Group D — Still unchecked

**14. Supply may be servicing gear during fake-quiet weeks** — *unknown*
`availability.py:4-8` and the Supply one-pager both state that Supply reads
the availability record Dispatch writes, and books maintenance when callout
load is lowest. Four responders now have artificially empty schedules. The
data to check this is not in the Dispatch database.

**15. Something we promised in 4.2 vanished** — *confirmed it vanished,
effect unknown*
Availability Confidence is committed to 4.2 on the roadmap, absent from the
release notes, not marked deferred, still owned by a PM who left in August.
An availability feature shipping alongside a timing change — whether that is
connected, nobody recorded.

### My read

The machine did not break. Every line did what it was written to do. We never
noticed that two sensible changes, combined with a seven-year-old shortcut,
would quietly remove four of our sixteen responders.

- **Items 1–4 are one bug, not four.** Fixing the timer reduces new victims.
  Fixing "miss ≠ refusal" stops creating them. Neither undoes the damage —
  **the four floored responders need their scores reset by hand**, because
  nothing in our code will ever do it for them.
- **Item 6 is the uncomfortable one.** Ten weeks of conversation have been
  about the half of the release I think is innocent. Reverting it — which
  Priya explicitly warned against — would undo a change responders asked for
  over three quarters and leave the real cause in place.
- **Items 10–13 are the ones that will repeat.** This was findable on day
  two. Marcus asked exactly the right question. Fix only the code and the
  same failure happens next release.

**Order I would work in:** talk to Wen Li (one conversation settles items 2,
3 and 6) → reset the four responders' scores → add delivery recording →
check the Supply knock-on → then decide about the timer.

---

## Detailed question set — 15 questions

### New evidence found here: the machine's stopwatch

Measured gap between one ping and the next, from the `pings` table:

| Situation | Before 4.2 | After 4.2 |
| --- | --- | --- |
| **Nobody answered** | **91–93 s** (avg 92.0) | **61–63 s** (avg 62.0) |
| **Responder tapped no** | 6–40 s (avg 22.5) | 6–40 s (avg 22.4) |

The 90-second and 60-second timers appear in the data to the second. The
refusal row is the control: human speed did not change. **We removed the
60-to-90-second window, which is exactly where the slow yeses lived.**

---

**1. What changed in 4.2, and what could affect selection, ranking or
delivery?**

Three constants in `config.py`, plus console work and three defect fixes. No
logic changed, no new code.

| Setting | Before | After | Affects |
| --- | --- | --- | --- |
| `OFFER_TIMEOUT_SECONDS` | 90 | **60** | Delivery outcome — hugely |
| `WEIGHT_PROXIMITY` | 0.45 | **0.60** | Ranking |
| `WEIGHT_RECENT_ACCEPTANCE` | 0.40 | **0.25** | Ranking |
| Console filter persistence | — | added | Noise only |

Notification delivery code was untouched. Push reliability was 4.1's work, in
June.

**2. How did the metrics move?**

| Metric | Before | After |
| --- | --- | --- |
| Acceptance rate | 76.6% | **64.0%** |
| Turned down | 21.1% | **18.0%** (improved) |
| Missed | 2.3% | **18.0%** |
| Pings per day | 24.7 | 23.6 |
| Pings per callout | 1.23 | **1.39** |
| Callouts needing 3+ pings | 12 | **26** |
| Nobody came at all | 5.5% | **11.1%** |
| Time-to-accept | — | **no column exists** |

Ping volume did not rise, so "too many notifications" is out. And we cannot
report time-to-accept — one of our three headline metrics has no field in the
database.

**3. Sudden or gradual?**

Sudden, and datable. 10 Aug 75.0% · 11 Aug 68.4% · **12 Aug 44.0%** · 13 Aug
40.0%. Missed calls: 1 on the 11th, 7 on the 12th, 12 on the 13th. The
stopwatch flips from 92 s to 62 s on the same boundary. This is a switch, not
a trend.

**4. Is everyone equally affected?**

No. Change in share of all pings:

| Lost most | | Gained most | |
| --- | --- | --- | --- |
| Farlight (Uptown) | −5.1 | The Gale (Eastgate) | +3.7 |
| Vesper (Old Town) | −5.1 | Stormwrack | +2.8 |
| The Undertow (Harborside) | −4.6 | Nightwell / Vantage / Falkirk | +2.7 |
| Meteor Mite (Eastgate) | −4.2 | Ironvale | +1.7 |

**The decisive test: Meteor Mite (−4.2) and The Gale (+3.7) both cover
Eastgate and share a handler.** Same area, busiest in the city, opposite
outcomes.

| Possible segment | Verdict |
| --- | --- |
| Location | **No** — two Eastgate responders at opposite extremes |
| Quiet area | **No** — Old Town is 3rd busiest of 15 |
| Low prior workload | **No** — Vesper was 2nd busiest overall |
| Poor performer | **No** — Vesper had the 2nd best acceptance rate, fewest refusals |
| Bad luck in week one | **Yes** — 4+ misses in the first 5 days, no exceptions |

**5. How does ranking work, and what changed?**

`score = 0.60 × closeness + 0.25 × recent yeses + 0.15 × skills`. Closeness =
`1 − (travel minutes ÷ 45)`. Recent yeses start at 0.5, +0.08 per yes, −0.12
per miss *or* refusal, never heals.

4.2 moved closeness up and recent-yeses down. Making recent-yeses count
*less* should have **softened** any spiral — a floored responder used to need
to be ~40 minutes closer to win, now ~19. The weight change was probably the
innocent half of the release.

**6. Is it contacting the same people repeatedly?**

Yes, measurably.

| | Before | After |
| --- | --- | --- |
| Share to the busiest 4 | 31.8% | **42.7%** |
| Share to the quietest 4 | 17.7% | **9.2%** |

**7. Does behaviour match the wiki?** No — see Question 1 above, items 1–3.
Also note the Glossary was last updated 4 August, before the release.

**8. Is it contacting unavailable, too-far or already-busy responders?**

- Unavailable: **no** — filtered before scoring (`routing.py:24`).
- Too far: **no** — beyond 45 minutes scores zero and ranks last.
- Already busy: **cannot tell.** 44 pings went out within an hour of that
  responder accepting another job (65 before 4.2 — proportionally similar).
  **We do not store job durations**, so this is unanswerable with our data.

Availability itself is only what a responder or handler typed in. Worth
noting *Availability Confidence* was a committed 4.2 item that vanished.

**9. Volume, frequency, timing, duplicates, delays?**

| Check | Result |
| --- | --- |
| Pings per day | 24.7 → 23.6 (down) |
| **Duplicate pings** | **Zero** — no responder got the same job twice |
| Gap after a miss | 92 s → 62 s |
| Gap after a refusal | 22 s → 22 s |
| Time-of-day | Even across 24 hours, no outage shape |

No flood, no duplicates, no delays. The only timing change is the deliberate
one.

**10. What is in the tickets?**

*Correction on the premise:* there is no "two unread feedback collections" in
our materials. What exists is 147 support tickets (database), 4 customer
interviews (wiki, September), and `00-rook/feedback/` which is empty.

| Theme | Before (44 days) | After (27 days) |
| --- | --- | --- |
| "No calls coming" | 1 | **26** |
| "Job vanished too fast" | 1 | **20** |
| Console filters | 3 | 13 |
| Supply / gear | 5 | 11 |
| Everything else | 30 | 37 |
| **Total** | **40** | **107** |

Two themes went from 2 tickets to 46, and both map to one line of code.

**11. What do tickets say that the dashboards miss?**

- Our headline metric fell for a *good* reason — refusals went down. A
  dashboard reading "acceptance down" looks like disengagement. It is the
  opposite.
- **"Nobody came" doubled** (5.5% → 11.1%) and is referenced nowhere at Rook.
  An emergency with no responder is more serious than any rate.
- Rates cannot show that four named people were removed. Ticket subjects can:
  *"Hardly anything for Farlight this week."*
- The silence is in no dashboard: Aunt Dot and Kip filed zero tickets.
- Time-to-accept cannot be reported at all.

**12. Ignoring, or not reaching?**

Neither. Refusals fell. And refusals land in **22 seconds**, unchanged — the
people who answer, answer fast. The ones we lost are those who needed 60–90
seconds to reach the phone, and we removed exactly that window. Misses rose
15.7 points, roughly the share of pings that were being answered in that
band.

Distinguishing delivery failure: for *"never goes off"*, **conclusively not
delivery** — no ping row exists. For *"gone too fast"*, not provable from our
data (no delivery column), but no outage pattern, no push code changed, and
handlers describe the phone buzzing.

**13. What is contradictory or missing across all four sources?** See
Question 1 of this file. The one that directly explains the deterioration:
our dictionary distinguishes *missed* from *turned down*; our code does not.

**14. Which code change lines up with the data?**

`config.py:9` — `OFFER_TIMEOUT_SECONDS = 60  # was 90 until 4.2`

| Evidence | Link |
| --- | --- |
| Gap after a miss | exactly 91–93 s before, 61–63 s after |
| Gap after a refusal | 22 s both periods — the control |
| Missed rate | 2.3% → 18.0% from 12 August |
| Tickets | 20 saying the job vanished before they could answer |
| Handler quote | "coat on, one boot on" (Mr. Ambrose, 13 Aug) |

The second link runs through `offer.py:30`: each extra miss becomes a −0.12
penalty → four responders floor out → ping share collapses → 26 tickets
saying "no calls coming." **Two lines of code produce both ticket themes.**

**15. Three most plausible root causes**

**#1 — Timer cut, amplified by "miss = refusal" and a score that never
heals.** *Very strong.*
For: the stopwatch fingerprint; the exact-date break; the four-miss threshold
with no exceptions; both ticket themes explained; the Eastgate pair ruling
out location. Against: nothing found.
To validate: ask Wen Li whether identical treatment was intended, and whether
anything resets the score.

**#2 — The weight rebalance.** *Weak as a primary cause, not excluded.*
For: it did change the order, and shipped the same day. Against: the
arithmetic says it reduces the spiral.
To validate: replay August's callouts with old weights and new timer, then
the reverse. That single test separates them — and needs Wen Li.

**#3 — August seasonality.** *Mostly wrong, but not entirely.*
For: incidents genuinely dipped, from ~138 a week in July to 110–127 in
mid-August. **Priya was not imagining it** — which is probably why the story
was so believable. Against: acceptance rate is a ratio, so fewer incidents
does not make responders answer less often; the first eleven days of August
were normal; and no season moves a stopwatch from 92 seconds to 62.
To validate: we would need a previous August. Rook does not have one.

### In one breath

We cut the answer window by a third. The people who say *no* were always
fast, so they were unaffected. The people who say *yes* need time to reach
the phone — and we took away exactly the half-minute they were using. Then we
punished them for it as if they had refused, with a score that never heals,
which stopped their phones ringing at all. Four responders fell in, and the
two whose handlers never complain are still in there.

---

## What I'd do next — the five steps, explained

| Step | What it costs | What it's worth |
| --- | --- | --- |
| 1. Ask Wen Li | An hour | Might change everything below |
| 2. Un-stick the four responders | A small fix | Four people get their work back |
| 3. Record whether the phone rang | A small build | We stop being blind |
| 4. Check their gear is safe | An email | Nobody gets hurt |
| 5. Decide about the timer | A whole release | Only safe once we understand it |

### Step 1 — Ask the person who built it

One hour with Wen Li, who built the ranking. Priya's handover: *"it's a
conversation with her, not a document — there isn't a good document."*

Three questions:

1. Did you mean a missed ping to count the same as a refusal? *(Marcus's
   14 August question, still unanswered.)*
2. Is there anything that resets a responder's score that I cannot see in the
   code?
3. Which half of the release did you expect to cause trouble?

**Why first:** costs an hour and no code, and her answers may change every
other step. Doing anything before this risks doing it twice.

### Step 2 — Un-stick the four responders

Vesper, Farlight, The Undertow and Meteor Mite have a score floored near
zero. Someone has to reset it **by hand** — `history.py:30` confirms nothing
in our code will ever do it.

**The important distinction:** fixing the timer protects *future* responders.
It does nothing for the four already in the hole. Two separate jobs.

**One honest unknown:** nothing in our materials shows an admin tool for
editing a score. That is question one for Wen Li — another reason step 1
comes first.

**Why second:** four real people lose work every day this is not done.
Farlight received zero calls in the first week of September. Everything else
here can wait a week; this cannot.

### Step 3 — Record whether the phone rang

Our `pings` table records who we buzzed and what happened, but never whether
the phone actually rang. So "the buzz never arrived" and "the buzz arrived
and he was ten seconds too slow" are **identical** in our data.

This is why Nadia could not explain two thirds of her tickets — the answer
does not exist anywhere at Rook.

**Why third:** it fixes nothing today, but without it we are blind the next
time, and we will argue for another ten weeks.

### Step 4 — Check their gear is still safe

One question to the Supply team: *have you been servicing these four
responders' equipment during what looks like their quiet weeks?*

Supply reads the availability record Dispatch writes and books maintenance
when callout load is lowest (`availability.py:4-8`; the Supply one-pager says
our changes land on their side "without any change on our side"). Our four
responders now look permanently quiet, so **their suits and boots may have
been taken away for servicing** because our broken ranking said they were not
working.

The data to check this is not in the Dispatch database. It is one email.

**Why fourth:** the only step where being wrong could hurt someone
physically — and it is cheap.

### Step 5 — Then decide about the timer

Back to 90 seconds, stay at 60, or something between? Genuinely hard: 60
seconds does reach an emergency faster when it works; 90 gives the person
coming down the stairs a fair chance. Both matter and they pull against each
other.

**Why last — and this is the real lesson:** if we fix "a miss counts as a
refusal" first, the timer matters far less. A responder who is slow once
loses that one job instead of every future job. **The painful part is not the
short timer. It is the punishment attached to it.** We may not need to change
the timer at all — and we will not know until steps 1–4 are done.

### The rule behind the order

**Cheap and fast first. Big and hard to undo, last.** Steps 1 and 2 cost
almost nothing and could both be done this week. Rook has spent ten weeks
debating step 5 — the hardest, most expensive, most arguable one — while
Farlight's phone sat silent.

### Also on the list

- **Take the deferred-items question to Helen.** What happened to
  Availability Confidence, and what else is still a Q3 commitment.
- **Write the routing description Priya asked for.** Most of it is in
  Question 2 of this file.

**One standing caveat.** The seasonal hypothesis is unsupported and
untestable, not disproven. Incidents genuinely dipped in mid-August; that
does not move an acceptance ratio, and no season moves a stopwatch from 92
seconds to 62.
