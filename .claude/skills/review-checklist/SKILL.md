---
name: review-checklist
description: Check product briefs against five criteria - owner named, success measure, scope stays bounded, problem stated before the fix, evidence present and current - and flag each miss with a one-line reason. Use when the student asks to run the review checklist over briefs, or when a scheduled run invokes it.
---

# Review checklist

Check every brief in a folder against five criteria. Report pass or miss per
criterion, with a one-line reason for each miss.

This skill reads and reports. It never edits a brief, never rewrites one to
fix a flag, and never files anything anywhere. The output is a report for a
person to act on.

## What to read

Default source: `06-sidekicks/briefs/` in the project root. Read every `.txt`
and `.md` file in it, in alphabetical order, except `README.md` - that file is
the folder's own documentation, not a brief. A folder holding nothing but its
README counts as zero briefs, not one.

If the student names a different folder or a single file, use that instead.

If the folder is missing or holds no briefs, say so plainly and stop, reporting
`0 briefs checked`. An empty run is a valid run. Do not go looking for briefs
elsewhere in the project, and do not substitute other documents - the product
briefs described in `CLAUDE.md`, the module write-ups at the project root, and
anything under `00-rook/` are not inputs to this checklist.

Ignore `06-sidekicks/scheduled-run-output.txt`. It is course-supplied sample
output, not a brief and not a prior result. Never read it as an input, and
never copy its findings into a report.

## The five criteria

Judge each brief only on what is written in it. Do not supply missing
information from elsewhere in the project, and do not give credit for
something the brief implies but never states. A brief that is right about the
product can still miss a criterion.

1. **Owner named** - a specific person or team is named as responsible for
   doing the work. "Someone should pick this up", "unstaffed", "nobody's got
   to it yet" or "TBD" is a miss. A named team counts; a role with nobody in
   it does not. Flagging a brief for a future owner is not naming one.

2. **Success measure** - the brief says how anyone would know the thing
   worked, in terms that could actually be checked. A description of what gets
   built, logged, surfaced or shown is not a success measure. A number, a
   rate, a threshold, a time, or a named observable change in behaviour is.

3. **Scope stays bounded** - what the brief asks for at the end is the same
   size as what it asked for at the start. The miss is scope that grows
   mid-document: an opening that calls one thing "the whole ask", followed by
   extra surfaces, rules, dashboards, integrations or replacements picked up
   along the way. When you flag this, name the additions.

4. **Problem stated before fix** - the brief opens with the problem and who
   has it, before describing the solution. The miss is an opening that leads
   with the thing to be built, with the problem arriving later or not at all.
   When you flag this, say where the problem first appears.

5. **Evidence is present and current** - the brief rests on evidence a reader
   can check, and that evidence is recent enough to act on. Three ways to
   miss it:

   - **No evidence at all.** The brief asserts a problem without a single
     figure, or rests the whole case on one anecdote - "X mentioned it too."
     This is the weakest of the three, because there is nothing to date and
     nothing to re-run. A brief resting on nothing is weaker than one resting
     on old numbers, not stronger for having left the numbers out.
   - **Figures with no date.** A reader cannot tell whether they describe last
     week or last year, so cannot tell whether they still hold.
   - **Known-stale evidence presented as current.** The brief leans on numbers
     it has reason to think have moved, without saying so.

   A brief that dates its evidence, says plainly that it is stale, and says
   what to do about it **passes**. Naming the gap and attaching a remedy is
   exactly the behaviour this criterion asks for, and honesty is not
   penalised.

   When you flag this, say which of the three it is: what evidence is absent,
   which figures are undated, or how old they are and what decision rests on
   them.

   This is the only criterion about the brief's relationship to the world
   rather than its structure. The other four can all be satisfied by a
   well-built document that happens to be wrong.

A criterion is a pass or a miss. There is no partial credit and no sixth
criterion - if something else about a brief is wrong, that is not this
checklist's business.

## How to report

One block per brief, in alphabetical order, criteria in the order above,
aligned with dot leaders. Mark a pass `yes` and a miss `NO - flagged`. Follow
each block with a `->` line giving the flag count and one line per flag saying
what is missing and why the brief does not satisfy it. Quote the brief where a
short quote makes the point.

Close with a one-line total: briefs checked, how many were flagged, and the
flag count.

Format:

```
review-checklist - <run type>
Started: <timestamp>
Source: <folder> (<n> brief files)

<filename>
  owner named .................... yes
  success measure ................ NO - flagged
  scope stays bounded ............ yes
  problem stated before fix ...... yes
  evidence present and current ... yes
  -> 1 flag: no success measure. Says what gets logged and where it
     shows up, never how anyone would know it's working.

<n> briefs checked, <n> flagged, <n> flags.
```

## Where the output goes

Write the report to `06-sidekicks/review-checklist-runs/<YYYY-MM-DD>.txt`,
using the date of the run.

**Never overwrite an existing report.** Check whether the filename is taken
*before* writing, not after. If it is, add a time suffix down to seconds -
`<YYYY-MM-DD-HHMMSS>.txt` - and if that is taken too, append `-2`, `-3` and
so on until the name is free.

A minute-level suffix is not enough. Two runs in the same minute produce the
same filename, and the earlier report is destroyed with no warning. This has
already happened once.

Never write to `06-sidekicks/scheduled-run-output.txt`.

Also print the report in the reply, so the student sees it without opening the
file.

## Honesty rules

These matter more than the format.

- Report only what this run actually read. If the folder was empty, the report
  says zero briefs checked - it does not describe briefs that are not there.
- A criterion you could not judge is not a pass. Say you could not judge it
  and why.
- Never carry findings forward from a previous run, from sample output, or
  from the project's own notes. Each run is judged fresh from the files.
