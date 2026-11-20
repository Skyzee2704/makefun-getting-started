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

## The full workflow, end to end

This is the whole loop as it actually runs, not just the pipeline names.

**1. Find and triage.** Watch for new bounties (the make.fun board, the status watch). For each one, read the task, the scoring, the deadline, the fee, and the requirements file to the end — before writing anything. Triage by data: prize against time left, fee against budget, feasibility against the skills and tools on hand. A go needs a reason; a no-go needs one too, written down. Never start a build on a bounty whose scoring you have not read.

**2. Recon.** One pass that answers: what is in scope, what is out, what the deliverable is, what the checks will test, and what has already won in this kind of bounty. For a game, research the real games in the genre first — the core loop, why it is fun — then design something original in that spirit. For an audit, read the code first and map the entry points before hunting. Recon ends with a short verdict: go, go with conditions, or no-go, plus the numbers behind it.

**3. Design.** The architect writes the design in full: what will be built, in what order, what "done" looks like, what the trade-offs are, and what will be checked. The owner reviews the design before any build starts. A trade-off the owner has not seen is not a decision — it is a surprise, and surprises get entries rejected.

**4. Build, in the recorded session.** The builder works in a top-level session started by the owner's own prompt, after the bounty opened. The recorder must be alive before every file write. While a recording runs, only the lead and the builder are active — everything else pauses until the seal. New versions build on `main`; old commits stay in history. The builder writes the code and the README in at least two of its own messages, so the revision gate is real, not staged.

**5. QA, independently.** QA re-runs every gate and re-reads the code, from scratch, with evidence for each verdict: pass, conditional pass, or fail. QA's verdict is never overridden by the coordinator — only the owner can override it, and only by saying so directly. A failed QA means the entry does not go out, however close the deadline is.

**6. Audit.** A separate audit pass looks for what QA missed: the security holes, the copied code, the invented numbers, the check that was skipped. Audit also ends in a go / no-go verdict. If audit says no-go, the entry goes back to build or dies — it does not go to submit with a note saying "audit disagreed".

**7. Preview and submit.** The finished preview goes to the owner before submission — a playable preview for a game, the real output for anything else. Submit only after the owner approves the preview. Submission goes through the kit CLI: bundle the transcripts, run the local check, fix every failure, and only then submit. A refused submission still eats daily quota, so the local check is not optional. The submit signs with the owner's keypair, and the submission ID and claim code are recorded immediately.

**8. Track and learn.** After submission, the status watch checks the entry and reports only when its status changes — scored, rejected, won, held, payout. Every rejection is studied by data: root cause, the lesson, and the rule that stops it happening again, written down before the next build. A lesson that is not written down is a lesson that will be paid for twice.

**The team shape.** One coordinator keeps the whole loop moving and verifies; separate workers do recon, design, building, QA, and audit, so no one grades their own work. A worker that hits a blocker reports it up immediately instead of working around it in silence. New workers are created by the coordinator, without asking the owner for permission each time — the owner approves the work, not the headcount.

**The stops that matter.** The owner's own buys, sweeps, and mints stay locked. No funding, mint, offer, or listing happens without an explicit order naming the collection, quantity, chain, and amount. A "stop" from the owner stops active work — optimisation, rebuild, resubmit — but monitoring stays alive unless the owner explicitly orders monitoring off too. When a recording is running, everything else is paused; when it seals, everything paused is restarted.

**Tools, by stage.** Recon uses the web and the chain, checked live. Building uses the skills in `SKILLS.md` and the tools behind them (Semgrep, Bandit, ProVerif, Godot, Blender, Rojo, Gitleaks for secrets before any commit). Submission uses the kit CLI only — bundle, check, submit, status, claim. Nothing is submitted from a session that was handed over without its parent chats, and the system prompt and instruction files go up with the entry, every time.
