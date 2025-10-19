# ScholarVerse

ScholarVerse is a SIP-009 compatible NFT marketplace and lottery smart contract for Stacks (Clarity). It provides NFT listing, direct buy, off-chain-style bidding/acceptance, royalty handling, and a simple admin-run lottery to reward participants.

## Key features
- SIP-009 trait compatibility for NFT ownership and transfers.
- List, cancel, buy NFTs.
- Place and accept bids (seller accepts a bid to transfer NFT).
- Royalty configuration per NFT contract (contract owner only).
- Integrated lottery: admin starts/ends rounds, users enter by paying an entry fee, winner selection, reward claiming.
- Persistent storage for listings, royalties, bids, lottery entries, winners, and rewards.

## Repository layout
- contracts/
  - scholarverse.clar — main contract implementing marketplace + lottery
- README.md — this file

## High-level design
- Listings: map token-id => { seller, price, nft }
- Royalties: map nft-contract => rate (percentage as uint)
- Bids: map token-id => { bidder, amount }
- Lottery:
  - entries mapped by round -> list of principals (max 100)
  - winners per round and reward balances per user
  - admin-controlled lifecycle (start/end)
  - entry fee and max entries configurable as constants

## Important functions (summary)
Marketplace
- list-token(nft-contract, token-id, price): list an owned token for sale.
- cancel-listing(nft-contract, token-id): remove a seller listing.
- buy-nft-token(nft-contract, token-id): buy a listed NFT, pays seller minus royalty.
- place-bid(token-id, amount): place or overwrite a bid above current bid and >= listing price.
- accept-bid(nft-contract, token-id): seller accepts a bid; contract transfers NFT and payments.

Royalties
- set-royalty(nft-contract, rate): contract-owner-only to set royalty percentage.

Lottery
- start-lottery(): admin opens a new round.
- enter-lottery(): user pays entry fee and is added to current round.
- end-lottery(): admin picks a winner using block-based seed, assigns reward.
- claim-reward(): winner claims STX payout.

Read-only helpers
- get-entries(), get-winner(round), get-reward-balance(user)

## Error codes / assertions (selected)
- u100: owner-only / admin checks
- u101..u109: listing, raffle, and bid-related errors (see contract constants for mapping)
- Ensure caller owns token when listing; ensure sufficient STX balances on buys/bids.

## Deployment & usage (quick)
1. Compile scholarverse.clar with Clarity tooling or via web wallet that supports contract deployment.
2. Deploy as the desired contract owner (contract-owner constant sets owner to tx-sender at deploy).
3. Use contract calls (list-token, buy-nft-token, place-bid, accept-bid) via wallet or scripts.
4. For lottery: set admin (var initialized to deployer), start-lottery, users call enter-lottery (pay entry fee), admin calls end-lottery, winner calls claim-reward.

Example (pseudocode)
- list-token: contract-call? scholarverse.list-token '(<nft-contract> 1u 1000000u)'
- buy-nft-token: contract-call? scholarverse.buy-nft-token '(<nft-contract> 1u)'

## Security notes & TODOs (important)
- Randomness uses burn-block-height modulo; this is predictable and insecure. Replace with verifiable/random oracle or commit-reveal for production.
- Validate STX transfer flows: seller/contract transfer ordering and use of as-contract may be incorrect in edge cases — review and audit payments.
- Bids currently replace prior bids without refund logic; extend to return previous bid amounts or hold bids in escrow.
- Add extensive unit/integration tests, gas checks and formal review before mainnet deployment.

## Testing
- Add Clarity unit tests (clarity-cli or Stacks testing frameworks) for:
  - Listing lifecycle, buy flow with royalties
  - Bid placement/acceptance and bidder refunds
  - Lottery lifecycle and reward claim edge-cases
  - Access control and error paths

---
This README is a concise guide. Update sections (constants, error codes, examples) to reflect any contract changes prior to publishing.
