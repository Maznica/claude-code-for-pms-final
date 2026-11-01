# 04 · X-Ray Vision — prompts

**Context:** You joined Rook two weeks ago as PM on Dispatch.
Release 4.2 shipped on 12 August, just before you arrived, and
landed badly. Last session the numbers showed you what happened: a
ping used to wait ninety seconds and now it waits sixty, people
missed pings they used to catch, and missing one counts the same as
turning one down — so four responders stopped hearing from us
altogether.

At the end of the session, type wrap up, and Claude Code saves the
prompts you wrote yourself below — not the starter prompt — updates
CLAUDE.md, and saves your work to GitHub. By Module 6 this file is a
prompt library built from your own questions.

---

### 1.

Let's go a little bit further - can you visualize the process alongside with the code files (.py files) so that I can understand it from the picture what is going on. 
In addition to that I have some questions:
Step 1 - you say nobody is filtered out, but actually these who are not available are filtered out, aren't they? Who decides about availability. How the code know who is and isn't available
Step 2 - We already know that the scoring algorithm was changing in the release. Can you explain in more detail the proximity - from my understanding above it might be just 1 or 0. Do I miss something here?
Where exactly in the code is that there is no difference between taping no and miss

### 2.

Who decides the how many minutes to the accident? How the code knows? You also said that there is another gate - wrong region? What does it mean exactly? I am asking because from the previous exercises I remember that also responders from other regions got calls

### 3.

Say your working theory out loud: the reason some responders are getting no pings at all is ___, because ___.

### 4.

So, how I understand it is that the acceptance score is from the history by old responders and by new one it is set to 0.5. So there is a slight difference between old ones and new ones. Can you scatter your table with respondents refused and pings in september

### 5.

Add all the insights from this session to module4_code_research.md

### 6.

It means, that there is nothing in the code which restores the score and the only way to increase the score is to tap yes, but when I don't get any calls I have no possibility to increase my rank.

### 7.

Before we wrap up I would examine 2-3 more things. 
First, create two charts: 1. Visualize the pings by the travel time before and after, where one circle is a ping. Color the pings by yes, no, miss, 2. do the same but just for the first three pings for each callout

### 8.

But you have to have knowledge about expected travel time (proximity) in minutes, because this is the only way how to compute the proximity score, right? So can you create these two charts with this expected travel time? I thought about some of scatter plots

### 9.

No, but I don't understand one thing. In previous exercises (especially in module 3) you computed the scores. How is this possible when the expexted time in minutes is not stored? In that way you won't be able to compute scores. In addition to that, what I found interesting that before release 4.2. no callout from neigbouring area was accepted (from your first chart) or do I understood something wrong.

### 10.

Is there in the code or database information that nobody went? I am asking because I remember that the nobody went rate also went up after the change. How is this possible? Is this driven by the fact that by all these callouts (where no one went) everybody had to say either no or miss. Can you visualize all these no went scenarios (one by one) and look at the pings by these cases by color no, miss. What I don't understand by this is that there are some responders with zero pings but still pretty high numbers of nobody went to. This would be only possible that I would set as no available. Do I miss here anything?

### 11.

Before we add this, let's elaborate on this even more? Can you find something in the data or code why the ranking list is so short in nobody came cases. The only thing I can think off is that someone sets me as not available. I disagree with you that you are excluded from the list when your expected travel time is larger than 45 minutes. From all the discussions, I understood that if this is the case your proximity score is zero but you still are part of the list. What I miss here?

### 12.

Before wrapping up let's do some simulation where we run what if analysis in one chart. 
1. first scenario is with current rules
2. Allow me  put different punish score for miss between 0 and 0.12 (maybe slider)
3. Score recovery to between 0.5 - 1 by the four responders without callout
4. Timeout change to 90 s

Create a chart, where I can do these what if analysis and the x axes is  the time (there is a clear line with the 4.2 release), y axis is the score. The lines are then the responders (the four with the lowest score), for the remaining eight drive median and lowest score (band)

### 13.

First, answer me the question below and then add to module 4 md file.
The score is still not the whole score but  just recent_acceptance_score, right?
