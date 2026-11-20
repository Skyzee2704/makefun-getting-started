# Kalos Drop Eligibility

## Treat every Kalos drop as Elysium mainnet

- Treat every Kalos drop as mainnet on Elysium chain 99901.

- Use https://rpc.elysium.kinetiq.xyz as the standing Elysium RPC.

- Never label a Kalos drop as testnet and never build a presign scheduler against a testnet RPC for a Kalos drop.

- Verify the chain id returned by the RPC matches 99901 before you read any on-chain state.

## Check eligibility from the Kalos API

- Fetch the drop list from GET https://api.kalos.fun/v2/launch/drops before you answer any eligibility question.

- Fetch per-wallet eligibility from GET https://api.kalos.fun/v2/launch/drops/{slug}/eligibility/{wallet} for each wallet the user names.

- Read stage times from the drop object itself and compare them with the eligibility response before you trust the eligibility result.

- Report eligibility per stage with its price and its maximum mintable count and never collapse stages into one word.

## Compute remaining allowance from the shared counter

- Read the on-chain mint count for the wallet across all stages of the same collection because SeaDrop counts mints in one shared counter.

- Compute remaining allowance as the stage maximum mintable by wallet minus the mints already counted in that shared counter.

- Never use balanceOf alone as the minted count because a wallet can hold tokens it did not mint through that stage.

- Treat remaining allowance of zero as definitively unable to mint for that stage even when the API still shows the stage as open.

## Arm a mint only when allowance remains

- Arm a presign or scheduler for a stage only after the live check shows remaining allowance greater than zero.

- Kill any presign or retry loop you find running for a stage whose remaining allowance is zero and report the stage and the process you stopped.

- Check that the wallet holds enough HYPE for the mint price plus gas before you arm anything and keep a reserve of at least 1 HYPE.

- Treat a zero price mint as still needing gas and simulate the mint before broadcast.

- Stop when simulation fails or the stage reports DONE or SOLD OUT or STAGE FULL.

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.

- If nothing here fits, look on the web for a maintained API or MCP server and set it up with [Custom Connectors and MCPs](/opt/muse/skills/custom-connector-setup/SKILL.md), or build a custom connector.

## Websites

- [Kalos](https://kalos.fun/) for the drop list and stage times. Carry the slug and wallet address into the check and verify the stage times against the API drop object.

## Alternatives for better results

- Ask the user which wallet and stage to check when either is missing rather than guessing a wallet or a stage.

- Re-run the live eligibility check immediately before a competitive mint because price and supply and allowance can change between check and execution.
