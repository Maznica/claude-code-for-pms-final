# 01 · Origin Story — prompts

**Context:** Rook Industries makes software for superheroes and the
people who handle them.

There are two products. Rook Dispatch is the one that gets a
superhero to where they're needed when there's an emergency — it
works out who is close enough and free enough to help, and gets hold
of them. Rook Supply keeps a responder's equipment serviceable and
accounted for, so a handler is never guessing whether the gear will
hold.

You joined two weeks ago as PM on Rook Dispatch. Release 4.2 shipped
on 12 August, shortly before you arrived. Responders have stopped
answering their phones the way they used to, and complaints have
gone up sharply. You were not in the room for any of it.

At the end of the session, type wrap up, and Claude Code saves the
prompts you wrote yourself below — not the starter prompt — updates
CLAUDE.md, and saves your work to GitHub. By Module 6 this file is a
prompt library built from your own questions.

---

### 1.

I will ask now three questions. Please answer them separately as detailed as possible:

1. What's contradictory or missing, not just in the company documents, but across Rook's wiki, its database and the files in my repo?
2. How exactly does the dispatch algorithm determine which responder gets contacted first, and how did that logic change in Release 4.2?
3. Is the decline in response rate caused by responders ignoring notifications, or by notifications failing to reach them?

### 2.

As you know I am new in the company and my role is product manager. So pretend you are product manager, answer the the first question and take into account only materials we have in the folder (don't halucinate), explain me the first question in a more simple english so that also 10 years child can understand it. In cases you can use some visualization, use and create also visualization. At the end also create a .md file with then name overview_questions.md

### 3.

Before we go further, still act as product manager, describe me the main product (which we try to fix) and take into account only materials we have in the folder (don't halucinate), explain me the product (describe customers and their painpoints, what problem do we want to solve, also the roles in the company we have, and how we solve the prblem) in a simple english so that also 10 years child can understand it. By answering you can use the skill /product-management:brainstorm  Besides text use visualizations.  At the end add the findings to the file overview_questions.md

### 4.

Before we move on explain me the relationship between handler and hero. I thought  we call here and he says yes/no. I don't understand the role of handler in in. Please visuzalize it and explain it in the way so that also a child can understand it. Take only into account the provided materials.

### 5.

Let's go more into the detail before we come to my questions, I would be interested in your analysis. Act as PM, analyze the problem and pick your 15 topics/areas with reasoning and some kind of likelihood, what might have gone wrong, use only materials we have (don't hallucinate), and explain it so that also a child can understand it (use also visualizations when appropriate)

### 6.

Before we wrap up, let's look in more detail into the following 15 questions (see below). Act as PM, answer the questions so that a small child can understand it, don't hallucinate, use also visualizations when appropriate. Questions:
What exactly changed in Release 4.2 compared to the previous version? Which changes could have affected responder selection, prioritization, or notification delivery?
How did the key performance metrics change before and after Release 4.2, particularly response rate, number of pings sent, time to acceptance, and failed contact attempts?
Did the decline in response rate begin immediately after Release 4.2, or did it develop gradually? Can we identify a specific point in time when system behavior changed?
Are all responders equally affected, or is the problem concentrated in specific segments based on location, availability, experience, workload, or contact frequency?
How exactly does the dispatch algorithm determine which responder gets contacted first, and how did that logic change in Release 4.2?
Has Release 4.2 caused the system to repeatedly contact the same responders while overlooking others? How has the distribution of pings across responders changed?
Does the system's actual behavior match what is documented in Rook's wiki? What inconsistencies exist between the documentation, the codebase, and the database records?
Is the system contacting responders who are actually unavailable, too far away, or already handling another emergency? How is real-time availability validated?
Did the volume, frequency, or timing of notifications change after Release 4.2? Are there signs of duplicate, delayed, or excessive pings?
What are the most common issues reported across the 147 support tickets and the two unread feedback collections? Do any recurring patterns correlate with changes introduced in Release 4.2?
What are responders telling us through support tickets that our aggregated performance metrics fail to capture? Are there important problems our dashboards do not measure?
Is the decline in response rate caused by responders ignoring notifications, or by notifications failing to reach them? Can we distinguish between these scenarios using database records and system logs?
What's contradictory or missing across Rook's wiki, support tickets, database, and codebase? Which inconsistencies could explain the deterioration in performance?
Which specific code changes introduced in Release 4.2 align most closely with anomalies in the pings table and patterns found in support tickets? Can we establish a direct connection?
What are the three most plausible root-cause hypotheses for the performance decline, what evidence supports or contradicts each one, and what additional data would help us validate them?

### 7.

You suggest this order: What I’d do first, in order: talk to Wen Li (one conversation answers items 2, 3 and 6) → reset the four heroes’ scores → add delivery recording → check the Supply knock-on → then decide about the timer.

What each of these steps mean exactly. Explain it like for 8-years child
