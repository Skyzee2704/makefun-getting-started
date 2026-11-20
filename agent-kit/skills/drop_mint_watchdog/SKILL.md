# Drop Mint Watchdog

## Arm every stage in one step

- List every stage that is armed for a future mint before you finish arming any one of them.

- Record each armed stage with its project, stage name, open time, price, and maximum mint count in one shared list that the scan and the watchdog both read.

- Add a new stage to the watchdog in the same step that you launch its presign, and treat a stage as unarmed until the watchdog covers it.

- Launch each presign and its retry loop as a standalone background process, and keep the retry loop separate from any worker tick so a slow worker start cannot delay the first retry.

- Never offer a manual mint as a fallback when a presign fails, and keep retrying through the script instead.

## Do not re-arm a stage the user cancelled

- Record a stage the user orders not to schedule in a do-not-arm list kept beside the shared armed list, with its project, stage name, reason, and the live proof behind the reason, such as the wallet already at its mint maximum or the supply sold out.

- Check the do-not-arm list before arming any stage on every scan tick, and never launch a presign or retry loop for a listed stage.

- Leave a listed stage off the armed list until the user explicitly orders it armed again or a fresh live check proves the reason no longer holds, and record which of those two lifted the entry.

- Kill a presign or retry loop you find running for a listed stage, and report the stage, the reason it stays off, and the process you stopped.

- Report a listed stage that is still mintable only once with its reason, instead of re-arming it silently on the next scan.

## Watch every armed stage

- Check the watchdog coverage against the shared armed list on every watchdog tick, and add any armed stage that is missing before its window opens.

- Derive the watched list from the shared armed list on each tick, and never rely on a fixed list of stage names that you wrote when the watchdog was first created.

- Treat a stage that opens within the next hour as urgent, and do not leave its recovery to the next hourly scan.

- Check that exactly one presign and one retry loop run for each armed stage, and remove any duplicate before it can send a second mint.

## Relaunch and verify

- Relaunch a dead presign and its retry loop from the main session when you can, and verify the new process is alive with its process id and a fresh heartbeat.

- Confirm the relaunched presign still targets the same stage, open time, and mint count, and confirm the retry loop still waits for the same open time.

- Retry a non-terminal failure in the same run and on later scans while the window stays open, and stop only when the stage reports DONE or SOLD OUT or STAGE FULL.

- Record a terminal outcome the first time you see it, and stay silent about later repeats of the same terminal outcome.

## Report the outcome

- Report a new drop with its stage, open time, price, and maximum mint count as soon as the scan finds it.

- Report every new presign and retry loop that you install, and report every mint success, failure, or first terminal outcome with its cause.

- Report a presign that died, the stage it served, and the new process id after relaunch.

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.

- If nothing here fits, look on the web for a maintained API or MCP server and set it up with [Custom Connectors and MCPs](/opt/muse/skills/custom-connector-setup/SKILL.md), or build a custom connector.

## Alternatives for better results

- Launch the presign from the main session when worker launched processes keep dying with no log trace, and keep the watchdog as the backstop rather than the only launcher.

- Shorten the watchdog interval when a stage opens soon, so a death is found minutes after it happens rather than at the next hourly scan.

- Sign close to the open time when the mint allows it, so a relaunched presign does not carry a stale signature into the window.
