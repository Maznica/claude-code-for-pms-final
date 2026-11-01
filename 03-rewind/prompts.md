# 03 · Rewind — prompts

**Context:** You joined Rook two weeks ago as PM on Dispatch.
Release 4.2 shipped on 12 August, just before you arrived, and
landed badly.

Last session you read four conversations and every support ticket
since 4.2 — and found the two piles did not agree.

At the end of the session, type wrap up, and Claude Code saves the
prompts you wrote yourself below — not the starter prompt — updates
CLAUDE.md, and saves your work to GitHub. By Module 6 this file is a
prompt library built from your own questions.

---

### 1.

Can you exactly describe the difference between turned down and missed? What is exactly the difference - please explain it also in the way that a 10 years child can understand it.

### 2.

So there is a clear problem with the missed rate, which might indicate the timeout problem (shorten time to catch). However, as we don't see if the ping was really delivered, it might be also a delivery problem (e.g. network issue), right? What do you think?

### 3.

Can you group the data by responder before and after in absolute numbers and shares pings, taken, missed, turn down to see some patterns.

### 4.

So, it seams that the data kind of confirms our hypotheses, right? We will now check your number against tickets:

### 5.

Can you rephrase the verdict so that it is more understandable. In addition, use also example so that a 10 years child can understand it

### 6.

So, it means that our story still holds and also the procedure: Ask Wen Li, put the scores to normal, write down whether the phone ring, check the gear and then decide about the timer. But maybe before deciding about the timer, we should change punishment scheme (just for no and not for miss)

### 7.

Can you visualize this data for Vesper before and after and also can you give me the score or rank of Vesper week by week so that I can see the patterns before and after

### 8.

If we go back to where we groupped responders by outcome and before and after 12 August. Can you visualize the data pings , yes, no, misses. Go both by week and also then by date

### 9.

Can you now take two responders from each group went quiet (here we already choose Vesper, so add to them another one and from the group got busier (also two responders).
Visualize the data before and after week by week, where one block = one ping, Color of the block is taken, turned down or missed. Write to the responder also the handler and region and add to the figure also the score, where score is replayed, end of the week. Make everything in one figure

### 10.

The figure is nice, can you just reorder the blocks (pings) so that they also shows exactly how the pings went in the time

### 11.

Add everything so far from this session into module3_data_research.md

### 12.

Can you visualize the map where the responders are located and where the incidents happen. Do this before and after in order to see how the proximity is important. My concern is that the proximity might cause also some issues why the people are not pinged.

### 13.

Use 3 examples and visualize it to back up following hypothesis:
"The change applies to everyone's existing score. But before 4.2, people's turn-downs had been wiped out by the 1.0 ceiling, so old history made almost no difference. After 4.2, turning jobs down didn't sink anyone either. What decided it was missed pings, because the code counts a miss as a turn-down. So the change didn't hit people who turn jobs down, old or new. It hit people who didn't answer within 60 seconds."
