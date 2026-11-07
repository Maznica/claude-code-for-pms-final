# briefs

Drop product briefs here as `.txt` or `.md` files, one brief per file.
This README is not a brief and is not checked.

The `review-checklist` skill reads every brief in this folder and checks it
against five criteria: owner named, success measure, scope stays bounded,
problem stated before the fix, and evidence present and current.

It runs every Monday morning on a schedule, and can be run any time by asking
for the review checklist. An empty folder is fine - the run reports zero
briefs checked.

Reports are written to `../review-checklist-runs/`.
