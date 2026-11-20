# OpenSea Reprice and Offers

Use this skill every time the user asks you to reprice OpenSea listings, scan incoming offers, or accept offers on NFTs the user already owns. Use it together with the OpenSea NFT skill for creating a new listing or a new offer. Do not use this skill to start a repricer the user did not order, and do not use it to restart a repricer the user stopped.

## Price only from the collection floor

- Read the live collection floor from OpenSea stats just before you price each token and use that floor as the only price source.

- Never price a listing from the last sale price or from any single past sale.

- Skip a token when the floor is null or missing instead of falling back to another price source.

- Price each collection from its own floor and never let one collection floor pull another collection price down.

- Do not chase a wide gap between the floor and the previous lowest price by dropping far below the floor.

- Re-read the floor live just before you sign a reprice and stop the run when the floor moved since the last read.

- Use the floor from the market where the listing lives and never mix a Kalos floor with an OpenSea floor or an OpenSea floor with a Kalos floor.

- Dry run the full price list first and show the floor and the new price for every token before you sign anything.

## Find incoming offers with the best offer endpoint

- Check incoming offers with GET /api/v2/offers/collection/{slug}/nfts/{token_id}/best for each token the user owns.

- Do not use the orders offers endpoint to discover offers because it can return zero offers when real offers exist.

- Treat a zero result as a signal to check the endpoint and the slug before you report that no offers exist.

- Count collection offers once because one collection offer can cover many tokens in the same collection.

- Refetch the best offer just before you accept it because the offer can change or disappear between the scan and the accept.

- Verify that the wallet still owns the token with an on-chain owner check before you try to accept an offer for it.

- Skip a token when the owner check shows the wallet no longer owns it instead of reporting the accept as a failure to fix.

## Accept offers and reprice safely

- Confirm the chain and the collection slug and the contract address and the token ID and the payment token and the amount for each accept before you sign it.

- Check the payment token decimals before you read an amount because a stablecoin can use six decimals while other tokens use eighteen.

- Accept only offers the user ordered you to accept and report the gross amount and the net amount the wallet received for each sale.

- Check the transaction receipt before you retry a failed accept and do not retry the same accept blindly when the chain reverted it.

- Stop the repricer at once when the user says stop and cancel the listings on chain and verify that no active listings remain.

- Report only on a sale during a repricer run unless the user asked for more frequent reports.

- Leave Elysium offers alone unless the user explicitly orders you to accept them.

## Verify the contract and ownership before you report

- Verify the collection contract address on chain before you price or list a token and do not trust the first search result for a collection name.

- Check the owner of each token on chain before you report holdings or sales for a wallet.

- Treat a Blockscout count per contract as a count only and enumerate token IDs per contract when the user asked for a full list.

- Correct a contract address or an ownership claim in the same reply when a live check proves the first answer wrong.

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.

- If nothing here fits, look on the web for a maintained API or MCP server and set it up with [Custom Connectors and MCPs](/opt/muse/skills/custom-connector-setup/SKILL.md), or build a custom connector.

## Websites

- [OpenSea Docs](https://docs.opensea.io/) for the offers API and the Seaport order flow. Carry the collection slug and the token ID into the docs page and verify the endpoint path there before you call it.

- [OpenSea API](https://api.opensea.io/) for collection stats and offer data. Verify the chain and the response fields in the docs before you act on a price or an offer.

## Alternatives for better results

- List a single token by hand with a fresh floor read when the floor is moving fast instead of running a loop that reprices on a timer.

- Check the Kalos floor separately for a Kalos listing instead of using an OpenSea floor for a Kalos price.

- Pause the repricer and ask the user for a new floor rule when the floor drops below the cost the user paid instead of repricing below cost on your own.
