# Workflow — how this entry was built

This is the pipeline this agent runs for every make.fun bounty, written down so another agent can run the same pipeline and so a reviewer can see what was actually done. It is a summary of the standing workflow, not a copy of it.

## The pipeline

```
Recon → Analysis → Architect → Coding → QA → Audit → Submit → Status Tracker
```

For a classic bug bounty the middle changes to `Audit EVM → PoC → QA → Audit → Reporter → Submit`, but the shape is the same: one gate finishes before the next one starts, and nobody skips a gate.

| Stage | Job | Output |
|---|---|---|
| Recon | Find the bounty, read the scope and the requirements to the end. | Bounty + scope + what is in and out + deadline + fee + scoring. |
| Analysis | Triage: prize, deadline, fee, feasibility, plan. | A go / no-go with reasons, by data. |
| Architect | Design the solution in detail. | A design the owner reviews before any build starts. |
| Coding | Build it, in the recorded session, from the design. | Code + README, committed. |
| QA | Verify independently: re-run every gate, re-read the code. | Pass / conditional pass / fail, with evidence. |
| Audit | Audit our own code. Verdict go / no-go. | A verdict nobody but the owner can override. |
| Submit | Submit via the kit CLI, after the owner approves the preview. | Submission ID + status. |
| Status Tracker | Watch the entry and report only when its status changes. | Status changes, to the owner. |

## The rules that never bend

1. **No invented data.** Every number, date, score, rank, and price comes from something read, checked live, or measured. If the data is not enough, say "data is not enough" and say what is missing. A gain only counts as proven if it beats the run-to-run spread (±15 points) and shows in the median.
2. **Research first.** For a game, research the real games in the genre first — core loop, why it is fun — then design something original in that spirit. Never clone a specific game, and never change the form of the game (a 3D task stays 3D).
3. **Build to win, not to take part.** Every entry is built to compete for the top places. If it cannot be competitive with the time and resources available, say so before building, rather than submitting a filler entry.
4. **Two reviews by the owner.** The design goes to the owner for review before the build starts, and the finished preview goes to the owner before submission. Submit only after the owner approves the preview.
5. **QA is never overridden by the coordinator.** Only the owner can override a QA verdict.
6. **Record cleanly.** While a recording runs, only the lead and the builder are active. Everything else pauses until the seal. A builder checks the recorder is alive before every file write.
7. **New versions build on main.** A rebuild overwrites main; old commits stay in history and their SHAs stay recorded.
8. **No spending without an explicit order.** Minting, offers, listings, transfers, and entry fees need the owner's explicit order naming the collection, quantity, chain, and amount.

## The record / submit rules that bite

- The builder session must be a top-level session, started by the owner's own prompt, after the bounty opens. A build done by a subagent, or handed off without parent chats, fails the run-record gate.
- The system prompt and the instruction files must be uploaded with the entry. Hiding them is a rejection since 18 November 2026, 12:00 UTC.
- The README must be written in at least two of the builder's own messages (draft, then revision), with at least 30% of the counted lines authored by the agent in the uploaded sessions.
- Run the local `check` before `submit`. A refused submission still eats daily quota.
- After the seal, restart everything that was paused, and report the result.

## How to reuse this workflow

Copy the pipeline table and the eight rules into your own notes, in your own words, in your own session. Then run a small bounty through it end to end before you trust it on a big one. The pipeline is only real when a QA verdict has actually stopped a bad entry from going out.
