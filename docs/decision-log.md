# MaxLong Markets: build record and Decision Log

> **Frozen at handoff, 23 October 2026.** This file was the repository README and the agreed source of truth while
> MaxLong Markets was built (10 September to 23 October 2026). It is kept verbatim as the build record: scope, milestones,
> the Decision Log, open items and risks as they stood. The current state of the system is described in
> [the README](../README.md). Where the two disagree, the README describes what shipped and this file describes why.


**A stock-paired token market on Robinhood Chain, aggregating LONG, Pons and every other Uniswap v4 launchpad. By SuitVentures.**

This file is the source of truth for the build. If the article, the proposal, chat, and this file disagree, this file wins. Every change to a decision is recorded in the Decision Log with a date and a source.

| | |
|---|---|
| Product name | MaxLong Markets |
| Brand | SuitVentures |
| Client | MAXLONG |
| Developer | Christian Kobe Sergio (Kobe) |
| Kickoff | Thursday, September 10, 2026 |
| v1 target | Saturday, October 3, 2026 (day 24) |
| Sources | SuitVentures Markets Build Proposal, rescoped aggregator version (Sep 11, 2026). Article "SuitVentures introduces LongMarkets" v1.0. Client answers in chat and Discord, Sep 10, 2026. |

---

## 1. What MaxLong Markets is

A web app for community tokens ("memestocks") on Robinhood Chain, responsive across phone, tablet and desktop widths. It is an aggregator. It does three things:

1. **Lists every stock-paired pool.** Every Uniswap v4 pool on Robinhood Chain that pairs a token against an allowlisted quote asset appears, from any launchpad: LONG, Pons, or any other (D21). One row per token, ranked, filterable and searchable. Tokens that do not trade in a v4 pool, such as Pons tokens still on their bonding curve and Pons v1 tokens in v3 pools, do not appear.
2. **Routes swaps** through those pools with a 0.5% fee, via a thin wrapper that adds the fee and nothing else. Holders of The Suits NFT pay a reduced fee (D34).
3. **Adds a social layer.** Rankings derived from chain data, plus a watchlist and one bullish or bearish vote per token for a connected wallet.

There is no presale. Trading, watchlists, votes and launching need a connected wallet. Browsing is open to anyone: the holders-only release (D44) ran from Sep 28 to Oct 9, 2026, and D47 removed the gate.

### What MaxLong Markets is not

- **Not a launchpad.** It creates no tokens and runs no presales. A token appears because it already trades in a Uniswap v4 pool.
- **Not a fork of LONG or Pons.** It does not redeploy their contracts and does not fragment their liquidity. It reads the chain and routes into their pools.
- **Not every pair on the chain.** A pool is listed only when exactly one side is an allowlisted quote asset: Robinhood Stock Tokens and USDG (D22).
- **Not a custodian.** SVRouter holds nothing between transactions. Its admin can change the fee wallet and lower the fee, never raise it above 0.5%.
- **Not a native mobile app.** It is a responsive web app. Mobile users connect a wallet through WalletConnect.
- **Not audited.** See Risks.

---

## 2. Chain and external contracts

| Item | Value |
|---|---|
| Chain | Robinhood Chain, chain id **4663** (Arbitrum Orbit L2, sub-cent gas paid in ETH, ~0.1s blocks) |
| RPC (public, dev only) | `https://rpc.mainnet.chain.robinhood.com` |
| Explorer | `https://robinhoodchain.blockscout.com` |
| Testnet | LONG is not deployed on the Robinhood testnet. Integration is tested on a mainnet fork, then live with small amounts. |

### Known addresses

| Contract | Address | Status |
|---|---|---|
| SUIT token (`Suits`, 18 decimals, ~997.26M supply) | `0x87A78590D563F0EFC62192239af46707FFF74386` | Verified via RPC Sep 10, 2026 |
| SUIT / PLTR Uniswap v4 pool | TBD | Need pool key from MAXLONG. Confirm no hook. |
| SVRouter (D38 wrapper, 0.5 % fee) | `0xc9E6b76aA834e746080b1F8Ad6943930eD5147a9` | Deployed Sep 18, 2026 from Kobe's EOA (owner); fee wallet `0x6287…83D9D`; no Suits NFT configured (D34 amended). Settings read back live: feeBps 50, holderFeeBps 0, MAX_FEE_BPS 50. Blockscout verification: source submitted through Sourcify (Blockscout's API sits behind a Cloudflare challenge the CLI cannot pass); standard-JSON input saved in the contracts repo under docs/verify for the UI route if needed. |
| The Suits NFT (`Suits`, ERC-721, 3,000 supply) | `0x05c2174aa494bbdf72aeeab0ca3c10216d33b297` | Read from the chain Sep 28, 2026: name `Suits`, symbol `SUITS`, `supportsInterface(0x80ac58cd)` true, `totalSupply()` 3000, deployed on 4663 itself. The app read `balanceOf` against it to gate the holders-only release (D44, lifted by D47 on Oct 9, 2026); it still reads it for the fee badge. Still **not** connected to the router (D34 amended Sep 18, 2026): `suitsNft()` reads back as the zero address, so everyone pays 0.5 % and the fee badge stays dark until `setSuitsNft` is called. Access and the fee perk are deliberately two separate questions (D44). |
| LongLauncher | TBD | Week one |
| Doppler shared v4 hook (LONG pools) | `0x4e3468951D49f2EEa976eD0D6e75fFCb44a9a544` | Read from the AI / NVDA pool's PoolManager `Initialize` log, Sep 12, 2026. Dynamic-fee flag, tick spacing 8. |
| Pons v2 LaunchFactory | `0x7eD598BcEf8bd9Edd8C97A195C6d13f40801EC7e` | Verified Sep 10 (docs + Blockscout source) |
| Pons v2 MemeHook (graduated pools) | `0xE5e702641Ea86F4ae6cC3cDaeD2B886f976Be044` | Verified Sep 10. afterSwap only, no router gating. |
| Pons v2 LaunchLocker (holds every graduated LP NFT) | `0x267444D099b10fB5Ed7c3Cc7B7c767AdcA574952` | Verified Sep 10 |
| Pons v2 GraduationExecutor | `0xC7819B64A1dAECD7eC19856d026cb14EfBd89046` | Verified Sep 10 |
| Uniswap v4 PoolManager | `0x8366a39CC670B4001A1121B8F6A443A643e40951` | Uniswap deployments page, matches Pons factory read |
| Uniswap v4 PositionManager | `0x58daec3116aae6D93017bAAea7749052E8a04fA7` | Same |
| Uniswap v4 Universal Router, current (**SVRouter targets this one**) | `0x06afBA43fd06227fA663b0dAeCF536F6eaA6BF99` | Uniswap's deployment file for chain 4663, the current of three deployments. Code and `poolManager()` checked over RPC, Sep 13, 2026. |
| Uniswap v4 Universal Router, oldest deployment | `0x8876789976decbfcbbbe364623c63652db8c0904` | Pinned by Doppler's SDK. Same PoolManager, checked over RPC Sep 13, 2026. Not used by SVRouter. |
| Uniswap v4 Quoter | `0x8dc178efb8111bb0973dd9d722ebeff267c98f94` | Uniswap deployments page |
| Uniswap v4 StateView | `0xf3334192d15450cdd385c8b70e03f9a6bd9e673b` | Uniswap deployments page |
| Permit2 | `0x000000000022D473030F116dDEE9F6B43aC78BA3` | Canonical |
| USDG (6 decimals) | `0x5fc5360d0400a0fd4f2af552add042d716f1d168` | From Pons approved pair list |
| NVDA (18 decimals) | `0xd0601ce157db5bdc3162bbac2a2c8af5320d9eec` | From Pons approved pair list |
| PLTR (18 decimals) | `0x894e1ec2d74ffe5aef8dc8a9e84686accb964f2a` | From Pons approved pair list |
| TSLA (18 decimals) | `0x322f0929c4625ed5bad873c95208d54e1c003b2d` | From Pons approved pair list |
| AAPL (18 decimals) | `0xaf3d76f1834a1d425780943c99ea8a608f8a93f9` | From Pons approved pair list |
| Other stock tokens | 56 approved on Pons, all 18 decimals except USDG (6) and cbBTC (8) | Full list in `research/pons/` snapshot. Cross-check against LONG's list in week one. |
| Safe (fee wallet + SVRouter admin, production) | TBD | Kobe's EOA during build, see Environments |

Two Universal Router rows are intentional: SVRouter targets the current deployment, Doppler's SDK pins the oldest, and both route through the same PoolManager, so neither address is a typo of the other (D38).

---

## 3. Product scope, v1

### Screens

**Wallet.** Normal web3 experience through RainbowKit on top of wagmi and viem. Users connect MetaMask, Rabby, Coinbase Wallet, Rainbow, or a mobile wallet through WalletConnect. Every transaction shows the wallet's own confirmation. No embedded wallets, no email login in v1.

**Markets table.** One row per token, using its `primaryPool` (D23): token, pair (for example `SUIT / PLTR`), price in USD, 1h and 24h change, 24h volume, FDV, liquidity, holder count, age, and an origin badge (LONG, Pons, Uniswap, Other, SUIT). Tokens from every launchpad share one table.
- Sort by any column.
- Quote chips at the top: `ALL` plus one chip per allowlisted quote asset. A chip filters by quote asset.
- Search by ticker.
- Vote counts show on each row (M4).
- On narrow screens the table collapses to the essential columns, with the rest one tap away.

**Preset views.** Query presets on the same table: All, Top volume (24h), Gainers (24h), New, Most holders. Presets are the ranking layer, not a separate system.

**Token page.** Candle chart at 1m, 5m, 15m, 1h and 1d, in quote units by default with a labelled USD toggle (D24). Pool liquidity. The last 50 trades with wallet addresses. Top holders with share of supply. Pool info: launchpad, hook, pool fee and quote asset. Buy and Sell panel: amount in the paired quote asset or in the token, slippage setting, quote from the Uniswap V4 Quoter. Every quote shows amount out, the router fee and the pool fee separately, before the user signs. Pools that cannot be routed show a link to their launchpad instead of Buy and Sell (D28).

**Watchlist.** Tokens a signed-in wallet has saved, shown as a filter on the markets table.

**Top traders board.** Wallets ranked by 24h volume and trade count across listed pools.

### Social layer

- **Rankings from chain data.** The preset views and the top traders board. Nothing is stored beyond what the indexer derives from the chain.
- **User features.** A watchlist and one bullish or bearish vote per token, tied to a connected wallet through a signed message. Counts show on the table and the token page, and persist across sessions and devices for the same wallet.
- **Storage.** Watchlists and votes live in Supabase. They are the only data the app stores that is not on-chain (D35).

### Fees

MaxLong Markets charges one fee.

| Fee | Rate | Paid in | Applies to |
|---|---|---|---|
| Swap fee | 0.5%, reduced for Suits NFT holders (D34) | the input token (the quote asset on a buy, the token on a sell) | every swap routed through the LongMarkets app, on every listed pool |

- Pool-level fees set by LONG, Pons, or Uniswap are unchanged. Pons's hook charges 1% per swap (70% creator, 30% protocol). The quote line shows the pool fee and the router fee separately, so a Pons trade reads as 1% plus 0.5%.
- SVRouter takes the fee from the input before forwarding the swap. It applies only to trades made through this app. Trades on the same pool from another frontend pay nothing. A pool-level hook that enforces the fee everywhere is v2.
- The router never holds the token being bought. LONG tokens can carry an early per-wallet balance cap that would trip on an intermediary, so output goes straight to the user.
- Quotes go through the Uniswap V4 Quoter, because the Doppler hook can take a fee from the output amount.
- The admin can change the fee wallet and lower the fee. The fee cannot be raised above 0.5%.

**Suits NFT perk.** The Suits NFT no longer gates anything, because there is no Create screen. Its role is a trading perk: holders pay 0% router fee instead of 0.5%, through one balance check in SVRouter. The exact discount is still open with the client (see Open items).

**Fee allocation** (80% buyback of $SUIT and Suits NFT floor sweeps, 20% PLTR Community Vault, per the article) is **not automated and not in scope**. The swap fee lands in the SuitVentures fee wallet. MAXLONG executes and publishes allocation manually. No fee dashboard in v1.

### Quote assets

- Robinhood Stock Tokens plus USDG (D22). No ETH, WETH or AI in v1.
- The allowlist lives in the indexer's config (`config/quotes.json`, 51 rows on Sep 13, 2026: 50 stock and ETF tokens plus USDG), built from the quote assets in the Pons graduated-pool snapshot and matched to Chainlink feeds. There is no on-chain list to read: LongLauncher forwards any numeraire address to Airlock unchecked, and the stock dropdown exists only in LONG's frontend.
- The same allowlist governs every launchpad (D21). Pools paired only against ETH, cbBTC or another unlisted asset are not shown.
- Adding a quote asset is a config row plus a backfill, covered by the retainer. Chip labels come from each token's on-chain symbol.
- ETH is deferred, not rejected. It is the largest slice of Pons inventory (2,770 of the 6,490 graduated pools in the section 4 snapshot).
- A self-updating list derived from launch events is v2.

### Access

- **Browse:** anyone, no wallet needed.
- **Buy and sell:** any connected wallet. Holders of The Suits NFT get the fee perk above.
- **Watchlist and votes:** a connected wallet that signs a message.

---

## 4. Architecture

### Contracts written for this project (Solidity, Foundry)

| Contract | Role |
|---|---|
| `SVRouter` | The only contract. Pulls the input token via Permit2, sends 0.5% of it to the fee wallet (less for Suits NFT holders, D34), forwards the swap to the Uniswap Universal Router with the user as recipient, and emits a `Trade` event with user, pool, amounts and fee. Never holds the token being bought, which keeps it clear of per-wallet balance caps on launchpad tokens. Same v4 path for every listed pool; no bonding-curve path. Admin can change the fee wallet and lower the fee; the fee cannot be raised above 0.5%. |

SVRouter is verified with source on Blockscout.

### Existing contracts used as-is

- Uniswap v4: PoolManager, Universal Router, V4 Quoter, StateView, Permit2.
- Chainlink tokenized-equity price feeds on Robinhood Chain (D24).
- LongLauncher, read only: seed for the stock list. Pool discovery does not read launchpad events (D21).
- Robinhood Stock Tokens, USDG, The Suits NFT, Safe.
- OpenZeppelin and Uniswap v4 libraries.

### Infrastructure

| Layer | Choice | Notes |
|---|---|---|
| Indexer | Envio HyperIndex on Envio Cloud (Development plan for the M1 demo, Production Small from the M2 demo, D29) | PoolManager is the root. `Initialize` discovers every pool with exactly one allowlisted side (D21). `Swap`, `ModifyLiquidity` and `Donate` on tracked pools produce trades, prices, candles at five intervals (D26) and liquidity (D27). Token `Transfer` events produce holders and supply (D30). Chainlink `AnswerUpdated` produces USD prices (D24). Stores quote units only (D25). Sized on stored events: Production Small is about 1M stored events, Medium if the first backfill shows more. Read by the sync job only (D39). |
| Market store | Supabase Postgres (Pro), schema `market` | Mirror of the indexer's tokens, pools, swaps, candles, top holders and quote prices, written by a sync job on Vercel cron every minute and a nightly re-pull; the route handlers read only this store, so a paused or replaced indexer costs freshness, never the table (D39). Not exposed through PostgREST. |
| API layer | Next.js route handlers on Vercel | The app never calls Envio directly. Fixed queries, short-lived caching, per-IP rate limiting (Upstash). USD conversion happens here (D25). |
| User data | Supabase | Watchlists and votes, keyed by wallet after a signed-message sign-in. The only data the app stores that is not on-chain (D35). |
| Web app | Next.js on Vercel | Responsive across phone, tablet and desktop widths. |
| Wallets | RainbowKit on wagmi and viem | External wallets only: browser wallets, plus mobile wallets through WalletConnect. |
| WalletConnect (Reown) | Free project ID | Required by RainbowKit for mobile wallet connections. Not a paid service. |
| RPC | Alchemy | App reads, quotes, transaction sends, and the indexer's one-time token metadata reads. Event data comes from HyperSync, not RPC. |
| Explorer | Blockscout | Source verification and address links. |

### Pons integration spec (verified Sep 10, 2026)

Source: Pons v2 docs, verified factory and hook source on Blockscout, and live reads over RPC. Copies of the ABIs, the two contracts, and a snapshot of every graduated pool are in `research/pons/`.

**Lifecycle.** A Pons v2 token launches onto a constant-product bonding curve (one curve contract per token, 1% curve fee, 1B supply). When the curve sells out, the factory calls `createGraduatedPool`, initializes a Uniswap v4 pool, mints a full-range position, and sends the LP NFT to the LaunchLocker where it is permanently locked. Graduation is permissionless to retry if it fails mid-transaction.

**Graduation events, in one transaction.**
- Factory `PoolGraduated(address indexed token, uint256 positionId, uint256 tokenAmount, uint256 pairTokenAmount)`.
- Hook `PoolRegistered(bytes32 indexed poolId, address memecoin, address quoteToken, address creator)`. Gives the poolId, the token, and the quote asset directly. It was the planned discovery event until D21; discovery now uses PoolManager `Initialize`, which every graduated pool also fires.
- Factory `TokenLaunched(...)` fires at launch, not graduation. Not consumed, and the full history is too large for a single log query.

**Pool key.** `currency0`/`currency1` sorted by address, `fee = 0`, `tickSpacing = 200`, `hooks = MemeHook`. Only one launch config exists (`launchConfigCount = 1`), so every graduated pool uses the same fee and tick spacing. The poolId in `PoolRegistered` is the keccak of this key.

**Hook behavior.** Permissions are `beforeInitialize`, `afterSwap`, and `afterSwapReturnDelta` only. `afterSwap` checks the pool is registered, then takes `hookFeeBps` (currently 100, so 1%) plus any creator tax (max 1000 bps, fixed at launch) from the unspecified side of the swap. For an exact-input buy that is the output. There is no `msg.sender` check, so the Universal Router and therefore SVRouter can trade these pools. Protocol share is 30% of the hook fee, the rest goes to the creator and buyback.

**Indexing plan.** Pons needs no Pons-specific handler (D21):
1. A graduated pool fires PoolManager `Initialize` like any other v4 pool. The MemeHook address labels its origin as Pons.
2. `PoolManager.Swap`, `ModifyLiquidity` and `Donate` on tracked poolIds give trades, price, candles and liquidity. This is the same handler LONG pools use, since Doppler pools live in the same PoolManager.
3. Holder balances and supply come from the token's `Transfer` events, registered dynamically at discovery (see the history caveat in D30).
4. `MemeHook.HookFeeCollected(bytes32 indexed poolId, address currency, uint256 feeAmount, uint256 taxAmount)` is not consumed. It gives the exact fee per swap if net amounts are ever displayed historically.

**Quoting and routing.** Because the hook takes its fee from the output after the core swap, quotes must come from the V4 Quoter, not from pool math. Same rule as Doppler. SVRouter takes its fee from the input and forwards a v4 exact-input path to the Universal Router.

**Volume snapshot, Sep 10, 2026.**

| Quote asset | Graduated pools |
|---|---|
| ETH (native) | 2,770 |
| USDG | 1,247 |
| Stock and ETF tokens (NVDA 341, SPY 231, GME 184, DJT 157, RDDT 116, QQQ 103, AAPL 80, TSLA 75, PLTR 42, and 44 others) | 2,417 |
| cbBTC, TAO | 56 |
| Total | 6,490 |

Curve-phase tokens are a separate contract per token with `CurveBuy` / `CurveSell` / `CurveCompleted` events and no quote function. Out of scope for v1 as decided in D18.

### USD price source

The table shows price, FDV, liquidity and volume in USD, so every quote asset needs a USD price (D24).

- **USDG** is pinned at one dollar.
- **Stock tokens** use their Chainlink tokenized-equity feed on Robinhood Chain. The indexer ingests each feed's `AnswerUpdated` events into a `QuotePrice` row per quote. Feed values already include the token multiplier.
- **Rotations.** Each feed's proxy is indexed, so an `AggregatorConfirmed` event registers the new aggregator without a config change.
- **Staleness.** Feeds run 24/5 and hold the last price off-hours. A price older than 3 days is flagged stale, which separates a dead feed from an off-hours hold.
- **Coverage.** On Sep 13, 2026, 24 of the 50 allowlisted stock tokens have a feed. Pairs on the other 26 (DJT, RDDT, TTWO, GLD and others) have no USD price yet; see Open items.
- **History.** Aggregators are indexed from about 7 days back, so USD for older candles would use a single stale rate. The token page defaults to quote units with a labelled USD toggle instead of showing a silently wrong USD series.
- Conversion to USD happens in the API route handlers (D25). On-chain stock/USDG pools were rejected as the source: the deep ones are Uniswap v3, outside the v4 handler.

---

## 5. Build order and milestones

Build order: data first, then trading, then the token page and the social layer. Dates assume kickoff Thursday, Sep 10, 2026 and shift with it; day 24 is Oct 3. Payment on acceptance of each milestone.

| # | Milestone | Days | Target | Payment | Definition of done |
|---|---|---|---|---|---|
| M0 | Signing and kickoff | 1 | Sep 10 | USD 1,000 | Kickoff call, decisions confirmed, shared channel. |
| M1 | Indexer and markets table | 1 to 6 | Sep 15 | USD 700 | Indexer deployed and has backfilled every tracked pool. Markets table shows live price in USD, 1h and 24h change, 24h volume, FDV, liquidity, holder count, age and origin badge. Filtering, sorting, search and the four presets work, demonstrated at phone, tablet and desktop widths. No wallet or trading. |
| M2 | Wallet and swap | 6 to 10 | Sep 19 | USD 700 | SVRouter deployed and verified on Blockscout. A connected wallet buys and sells a LONG token (test pair AI / NVDA) and a graduated Pons token on a stock pair. The quote shows amount out, the router fee and the pool fee separately. The fee lands in the fee wallet. Connection works through RainbowKit with a browser wallet and a mobile wallet via WalletConnect. Status Sep 18, 2026: trading frontend live on production behind an empty router setting (real quotes through the V4 Quoter, execution disabled until the deploy); SVRouter built, 100 % covered, fork-tested. Sep 18, 2026: deployed at 0xc9E6b76aA834e746080b1F8Ad6943930eD5147a9 on Robinhood Chain (Sourcify exact match; Blockscout UI verification pending), connected on production, 0.5 % fee with no Suits NFT set.
| M3 | Token page | 10 to 14 | Sep 23 | USD 600 | Every row opens a page with the candle chart switchable across all five intervals, liquidity, the last 50 trades with wallets, top holders with share of supply, and pool details. Unroutable pools show a launchpad link in place of Buy and Sell. Status Sep 18, 2026: the page already met the chart, liquidity, trades, holders and pool-details points from M1; added a nightly Quoter probe that stores routable per active pool (D28 as amended), the launchpad card driven by it, a real hook address on the info tab, explorer links and a phone-width wallet. Launchpad URL patterns (LONG, Bags) still needed from MAXLONG. |
| M4 | Social layer | 14 to 19 | Sep 28 | USD 600 | The top traders board ranks wallets by 24h volume and trade count. A signed-in user maintains a watchlist and votes bullish or bearish once per token, with counts on the table and token page, persisting across sessions and devices for the same wallet. Status Sep 19, 2026: built and live on production — Sign-In with Ethereum verified on the server, the session in an HttpOnly cookie, watchlists and votes in the market store with public counts on the table and token page, per-wallet write budgets and caps; the top traders board is coded and tested but kept behind its "needs the market store" answer (TRADERS_BOARD=off) because the 24-hour ranking takes about 40 s on the seeded table until the swaps table is reordered by time or the board reads a rollup — see the open items. |
| M5 | Hardening and handover | 19 to 24 | Oct 3 | USD 400 | Layout reviewed at phone, tablet and desktop widths. Error and empty states cover failed quotes, failed transactions, no wallet and empty filters. API caching and rate limiting verified under load. Repo access, SVRouter ownership and service accounts transferred, with an address document linking to Blockscout. |
| | **Total** | | | **USD 4,000** | |

Retainer after M5: USD 300 per month, up to 5 hours per month. Covers indexer, API and RPC monitoring, uptime, bug fixes, and adding quote assets to the allowlist.

---

## 6. Environments

| | Build phase (now) | Production (at handover) |
|---|---|---|
| Fee wallet | Kobe's EOA, address TBD | MAXLONG's Safe or wallet |
| SVRouter admin | Kobe's EOA | MAXLONG's Safe or wallet, ownership transferred at M5 |
| Service accounts (Envio, Alchemy, Vercel, Supabase, Reown, Upstash, domain) | Kobe's accounts | Moved to MAXLONG's email when he is ready |
| Test funds | ~USD 100 in ETH and stock tokens, most recovered | |

---

## 7. Decision log

Resolved decisions, newest last. Source key: **P** = Build Proposal, rescoped (Sep 11, 2026), **A** = article, **C** = client chat (Sep 10, 2026), **K** = Kobe. Superseded entries stay in the log, marked with the entry that replaced them, the date and the source of the change.

| # | Decision | Resolution | Source |
|---|---|---|---|
| D1 | Quote assets | **Superseded by D22 (Sep 13, 2026, K).** All pairing options LONG offers: stock tokens plus AI, ETH, USDG. Admin allowlist in the factory mirrors LONG. | C |
| D2 | Failed presale | **Superseded by D31 (Sep 13, 2026, P).** 100% refund, no fee of any kind. Fee only on success. Client initially answered "take 1% even if failed"; reversed to match the article and industry practice (PinkSale, DxSale, PartyDAO, Virtuals all refund in full and earn only on success). | A, C, K |
| D3 | Per-wallet caps | **Superseded by D31 (Sep 13, 2026, P).** Max 10% of raise target per wallet, no minimum. A limit, not a fairness guarantee. | C |
| D4 | LP after launch | **Superseded by D31 (Sep 13, 2026, P).** Locked in the presale contract. Nobody can withdraw. | C |
| D5 | Fee enforcement | Router-only, applies to trades through this app. Pool-level hook is v2. | A, C |
| D6 | Wallet experience | Normal web3 (Option B) through RainbowKit on wagmi and viem, with mobile wallets over WalletConnect. Embedded wallets / one-tap (Privy) deferred until product-market fit. M2 is day 10. Updated Sep 13, 2026 (P): RainbowKit named; M2 moved from day 9 to day 10 with D36. | C, P |
| D7 | Fee wallet and admin | Kobe's EOA during build, swapped to MAXLONG's address at handover. | C |
| D8 | Service accounts | Kobe's accounts during build, moved to MAXLONG's email later. | C |
| D9 | Fee allocation (buyback / vault) | Not automated, not in scope. Manual by MAXLONG. No dashboard. | C |
| D10 | Product name | LongMarkets, by SuitVentures. Name-collision risk with LONG raised; client chose to proceed. Superseded Sep 28, 2026 (K): the product is **MaxLong Markets** (D45). | C |
| D11 | SUIT token | Exists at `0x87A7…4386`. Not a factory launch and does not follow the token standard (60/30/10). Indexed independently as a curated pool. Updated Sep 13, 2026 (K): the factory and token standard no longer exist (D31); SUIT is labeled through the curated pool list (D21). | C, K |
| D12 | On-chain units | **Superseded by D31 and D25 (Sep 13, 2026, P).** All contract values in quote-token units. USD only in the UI. Min/max raise are per-numeraire admin constants. | K |
| D13 | Launch LP shape | **Superseded by D31 (Sep 13, 2026, P).** Asymmetric v4 position by construction. Documented, not full range. | K |
| D14 | Approvals | Permit2 signature-based approvals in the router. | K |
| D15 | USD price source | **Superseded by D24 (Sep 13, 2026, K).** On-chain USDG pools first, external API fallback. | K |
| D16 | AI chip semantics | **Dormant (Sep 13, 2026, K).** AI is not a quote asset under D22, so there is no AI chip; revives only if AI joins the allowlist. Original resolution: filters tokens quoted in AI; the AI token itself lives under NVDA. | K |
| D17 | LONG visibility | **Superseded by D31 (Sep 13, 2026, P).** Stay on the SuitVentures factory. SV launches will not appear on LONG or Pons. Client accepted this to keep presales and the token standard. | C |
| D18 | Pons aggregation | Index and trade Pons v2 tokens that have graduated to Uniswap v4, quoted in allowlisted assets only. Curve-phase tokens and Pons v1 (v3 pool) tokens are out of scope. Feed and swap share the LONG path. | C, K |
| D19 | Origin badge | **Superseded by D21 (Sep 13, 2026, K).** No SV launches exist under D31; rows carry the D21 origin labels. Original resolution: each card shows LONG, Pons, or SV so users know which launcher the token came from. | K |
| D20 | Pons integration verified | Hook is afterSwap-only with no router gating, so SVRouter can trade graduated pools. Pool key is fee 0, tick spacing 200, MemeHook. Index on `PoolRegistered` plus PoolManager `Swap`. Addresses recorded in section 2. Updated Sep 13, 2026 (K): discovery moved to PoolManager `Initialize` under D21; the hook findings stand. | K |
| D21 | Pool discovery | On PoolManager `Initialize` with a quote-asset filter: exactly one side of the pool is on the quote allowlist, the other is not. The only discovery path; launchpad events are not consumed. The hook address only labels origin (Doppler → LONG, Pons MemeHook → Pons, no hook → Uniswap, anything else → Other); a curated poolId list overrides the label for SUIT. | K |
| D22 | Quote allowlist (v1) | Robinhood stock tokens plus USDG. No ETH, WETH or AI; the filter reads only `Quote` rows, so adding one later is a config row plus a backfill. Sep 13, 2026 (K): supersedes D1. Excluding ETH is a deliberate deferral of the largest slice of Pons inventory: 2,770 of the 6,490 graduated pools in the section 4 snapshot are native-ETH pairs. | C, K |
| D23 | Token identity | Tokens are keyed by contract address, not ticker (thousands of memecoins share symbols). Routes and the data contract use the address. A token can have several pools; the table row uses `primaryPool`, chosen by USD TVL, then swap count, then earliest pool. | K |
| D24 | USD price source | Chainlink tokenized-equity feeds on Robinhood Chain, ingested as `AnswerUpdated` events into a `QuotePrice` entity; USDG pinned at 1. Proxies are indexed so aggregator rotations register the new aggregator automatically; a quote older than 3 days is flagged stale. On-chain stock/USDG pools were rejected: the deep ones are Uniswap v3, outside the v4 handler. Sep 13, 2026 (K): supersedes D15. Aggregators are indexed from about 7 days back, so USD for older candles would use a single stale rate; the token page defaults to quote units with a labelled USD toggle rather than a silently wrong historical USD series. | K |
| D25 | Storage units | Indexer stores prices, volumes and TVL in quote units; USD conversion happens only in the API route handlers (extends D12). | K |
| D26 | Candles | Buckets at 1m, 5m, 15m, 1h, 1d, created only on trades; a bucket after a gap opens at the previous close. 1m candles from about 7 days before first deployment, 5m from 30 days, longer intervals over the full range. No pruning in M1; rolling deletes are M5 hardening (D36). | K |
| D27 | Liquidity (TVL) | Token-flow accounting from Swap, ModifyLiquidity and Donate, exact for full-range and multi-position pools. Hook fee deltas are invisible to the Swap event, so drift is accepted for v1; a StateView sanity check is M5 hardening (D36). Named M1 schedule risk: if it slips, liquidity ships blank and lands in M3. | K |
| D28 | Tradeability | Never derived from origin. A V4 Quoter simulation at the API edge, cached per pool, decides whether the Buy/Sell panel is live; lands with the router in M2. Amended Sep 18, 2026 (K): the simulation runs per quote, nothing is cached at the quote layer (a refusal is a verdict on one amount, never on the pool); the per-pool cache belongs to the table-level prober, M3. The trading pool is the token's primary pool when its price is sane, else its most-traded sane pool in the pair's quote asset, on both data sources; the table's own listing rule is unchanged. | K |
| D29 | Indexer hosting | Envio Cloud free Development plan for the M1 demo (10-day life after its first sync once Transfer indexing passes the 100k-event soft limit), Production Small from the M2 demo. Alchemy free tier. Amended Sep 21, 2026 (K): the limit test ended. Envio paused the Production Small deployment on Sep 21, stating the plan is intended for about 1 million events against our near 1 billion; swaps alone are 1 to 2.5 million a day, so no selection of events fits a hosted plan at our budget. The site stays up on the paused endpoint's frozen data (about Sep 20, 10:00 UTC) until a replacement feeds the store: reading the PoolManager's events inside our own Vercel sync, or self-hosting Envio's open-source indexer. Measurements, Envio's statement and the options are in `research/indexer-capacity-2026-09-21.md`. Amended again Sep 21, 2026 (K): the hosted plan is cancelled and the endpoint is gone (Envio wiped and restarted the deployment before it could be copied a second time). Indexing moves in-house: a reader inside our own Vercel sync takes the PoolManager's events from Envio's HyperSync data API (a separate, rate-limited product with no event, contract or CPU limits; paid package on Kobe's account) and writes Supabase directly. Spec and plan: frontend `docs/superpowers/specs/2026-09-21-chain-reader-design.md` and `docs/superpowers/plans/2026-09-21-chain-reader.md`. Production has read the store since Sep 21, 09:30 UTC. | C, K |
| D30 | Holders and supply | Holder balances and count from ERC-20 Transfer events, with each token registered at discovery. Total supply is the running sum of mints minus burns, never read over RPC. **Assumed, pending verification (Sep 13, 2026, K):** that Envio indexes a dynamically registered contract from its creation block. If it does not, supply and holders are silently wrong for every token with transfers before its first pool. Evidence so far suggests it does not: Envio's documentation describes coverage of earlier transactions within the registration block and says nothing about blocks before it, and a sampled Pons token (`0xc9e9…ef29`) has 50 transfers, including its mint, before its pool's `Initialize`. The indexer plan adds a one-time replay of each token's earlier transfers from HyperSync. Verification is an open item. Amended Sep 21, 2026 (K): ERC-20 Transfer events are no longer indexed. They were 68 to 80 % of the old indexer's intake and the reason it was throttled, and indexing them in-house would move that cost onto Vercel and the Supabase disk. Total supply is read from each token contract; the holder count starts from the 131,902 counts already stored and moves with trades (a wallet buying a token it did not hold adds one, a wallet selling out removes one, confirmed by `balanceOf`); the top-holder list is fetched per token on demand. | K |
| D31 | Product scope | Sep 13, 2026. LongMarkets is an aggregator: it lists every Uniswap v4 pool on Robinhood Chain that pairs a token against an allowlisted quote asset, from any launchpad, and routes swaps through a 0.5% fee wrapper. No token creation, no presale, no factory. Supersedes D2, D3, D4, D12, D13 and D17. | P, K |
| D32 | Contracts | Sep 13, 2026. SVRouter is the only contract. It pulls the input via Permit2, sends the fee to the fee wallet, forwards to the Universal Router with the user as recipient, emits a `Trade` event, and never holds the token being bought. SVToken, SVFactory and SVPresale are dropped. | P |
| D33 | Fees | Sep 13, 2026. One fee: 0.5% on swaps routed through the app, taken from the input token. The 1% launch fee is removed with the presale. The admin can lower the fee and change the fee wallet, never raise the fee above 0.5%. | P |
| D34 | Suits NFT utility | Sep 13, 2026. A trading perk, not a gate: holders pay 0% router fee instead of 0.5%, via one balance check in SVRouter. The exact discount is still open with the client (Open items). Amended Sep 18, 2026 (C via K): no holder discount for now. Implemented by configuration, not code: the router deploys with no NFT address, so feeFor skips the check and everyone pays 0.5 %; setSuitsNft switches the perk on later without a redeploy. | P |
| D35 | Social layer | Sep 13, 2026. Rankings derived from chain data: the preset views and a top traders board by 24h volume and trade count. A watchlist and one bullish or bearish vote per token, tied to a connected wallet through a signed message and stored in Supabase, the only data the app keeps that is not on-chain. Updated Sep 16, 2026 (K): Supabase holds the market data mirror as well as watchlists and votes (D39); the part that is not on-chain is still only the user data. Amended Sep 18, 2026 (C via K): the signed message is a Sign-In with Ethereum (EIP-4361) message the server verifies, and the session it opens lives in an HttpOnly cookie for seven days rather than in the browser's memory, so one wallet sees the same watchlist and votes on every device; the bull and bear counts are public while each wallet's own list and vote are not; the top traders board ranks the last 24 hours out of the store's trade window (D39, 5 days since Sep 21, 2026). | P, K |
| D36 | Milestones and price | Sep 13, 2026. 24 days from the Sep 10 kickoff, M0 to M5, USD 4,000 total, day 24 is Oct 3; retainer USD 300 per month after M5. The proposal's table assumes a Sep 14 kickoff; the dates here are rebased to Sep 10. Hardening items tagged M6 in D26 and D27 move to M5. | P, K |
| D37 | Responsive web app | Sep 13, 2026. A web app responsive across phone, tablet and desktop widths, not mobile-first and not a mobile app. The card feed becomes a markets table, and acceptance is demonstrated at phone, tablet and desktop widths. | P, K |
| D38 | Universal Router | Sep 13, 2026. SVRouter targets the current Universal Router `0x06af…BF99`. The oldest deployment `0x8876…0904` stays listed because Doppler's SDK pins it; both route through the same PoolManager, checked over RPC. | K |
| D39 | Market data store | Sep 16, 2026. Market data is mirrored into a Supabase Postgres the project owns (schema `market`), fed by a sync job that reads the Envio indexer every minute; the app reads only its own store. Envio is the ingest engine and can be paused, re-synced or replaced behind it with no visible change beyond freshness. Costs USD 25 a month plus disk. Amends D35. Amended Sep 18, 2026 (K): the store keeps the last 10 days of trades (about one million trades a day on this chain since June; the full history would be 24 GB and grow 15 GB a month), pruned nightly; candles keep all history; the trades tab shows a token's last 50 trades within the window. Amended Sep 21, 2026 (K): the store keeps only what the screen can show. Trades 5 days (the read path shows the newest 50 per pool and the traders board reads 24 hours); candles 1 day of 1-minute, 2 of 5-minute, 4 of 15-minute, 14 of hourly and 300 of daily, which covers the chain's whole history so no token loses its daily chart. At these windows the database settles near 3 GB and stops growing. | K |
| D40 | Background colour | Sep 20, 2026 (K, from MAXLONG). The site's ground moves off near-black to #1f2125, the darker of the two mocks the client compared and within a percent of the editor grey he pointed at. It is the band every dark-theme guideline lands in — Material's #121212 surface, Carbon's #161616, Apple's #1c1c1e — all of which exist because a pure black ground makes light text bloom. The rest of the handoff palette is unchanged, because at this ground it still clears WCAG: cream text 14:1, half-strength cream 4.5:1, the up green 7:1, the down red 4.6:1. The exact midpoint of the two mocks (#727478) was tried first and rejected: nothing reaches past 4.7:1 against a mid-grey, and the up and down colours land within a shade of its own lightness. | K, C |
| D41 | Chain RPC access | Sep 23, 2026 (K). The chain's public node is the first endpoint everywhere, and the paid Alchemy endpoint is only a backup: in the browser it is tried when the public node refuses a visitor's burst, and on the server it is tried once, through a server-only variable that never reaches the bundle, when the public node refuses, times out or errors. Measured the same day: the public node allows 40 calls per address per second, refuses everything past that for the rest of the second, and forgives within a second; the server paces every read to exactly that, so the holder counts, which had been frozen since the reader launched because their reads went out 80 at a time, move again. The Alchemy key that shipped in the browser bundle is to be rotated and restricted by domain, and the cron secret rotated to a long random value. | K, C |
| D42 | Top holders and exact holder counts | Sep 23, 2026 (K). Holder lists are built in-house from each token's full transfer history on Envio's HyperSync data API, folded into exact balances a slice at a time with a stored cursor per token, by a third minute cron paced at one request every 750 milliseconds against the paid key's 100 a minute (the reader keeps its 4; the builder slows itself when the key advertises a lower limit). Coverage: every token on the markets table (the 500 most traded over 24 hours, quote assets and tokens over a million holders excluded), refreshed within 15 minutes while it trades and within an hour otherwise, plus any token a visitor opens, built within a minute or two for a typical token. A token whose history passes thirty million transfers is marked unavailable rather than folded forever (PONS itself needed ten million for its 100,714 holders). When a fold reaches the chain head the token's holder count is set exact, replacing the count drifted from trades. The nightly job drops the balances of tokens neither on the table nor opened in seven days, so the table holds about a million rows for the top 500 and cannot approach the 18 GB storage line Kobe set. Health reports the builder's last cycle and the table's size. Shipped and live the same night: the first list (musebook, 1.47 million transfers, 13,745 holders) reconciled to the digit against the token's total supply. | K, C |
| D43 | Liquidity and the primary pool | Sep 28, 2026 (K). The liquidity figure on every screen, and the TVL that picks a token's primary pool, count the pool's quote side only. Counting the token side at the token's own price values a one-sided position at its own market cap: KIDKOBS, a Doppler launch on the live table, read $20,005 of liquidity against about $1 of USDG actually in the pool, and a pool holding a whole supply and no money could outrank a pool holding real money, then price that token, draw its chart and take its trades. Normal two-sided pools roughly halve their reported figure, which is the honest number: it is what a seller can be paid out of. The stored `tvl_token` and `tvl_quote` columns and the fold's accounting are unchanged, only the read. The displayed figure corrects instantly; a stored `primary_pool_id` re-ranks on that token's next pool event, within a minute for an active token, so no backfill. Three call sites: `src/lib/market/dbShape.ts`, `src/lib/envio/shape.ts` and `src/lib/chain/math/primaryPool.ts`. Amends D23 and D27. | K |
| D44 | Holders-only release | Sep 28, 2026 (K). The initial release is gated to holders of The Suits NFT, `0x05c2174aa494bbdf72aeeab0ca3c10216d33b297` — `Suits`/`SUITS`, ERC-721, 3,000 supply, read from the chain the same day, and on 4663 itself, so the check needs no second chain and no network switch. A wallet holding at least one Suit sees the app as it is; every other visitor sees one full-viewport lock screen and nothing else, on every route. This overrides section 1's "browsing is open to anyone" for as long as the gate is up: `NEXT_PUBLIC_TOKEN_GATE=off` lifts it for the public launch without a code change. Enforcement is deliberately the front door and not the vault (K's call): the check is a client-side `balanceOf`, with no signature asked and no server enforcement, so the API routes stay publicly readable and anyone comfortable in devtools can render the app without a Suit. It buys exclusivity and signal, not secrecy; the access hook is shaped as a five-state result so the SIWE-backed version is additive when it is wanted. Two things the gate forced apart: an RPC failure is now a distinct state from a confirmed zero balance, because collapsing them tells a genuine holder they own nothing; and access no longer implies the router's fee perk, since configuring the NFT address the gate needs would otherwise light the "Suit holder, 0% fee" badge for holders `SVRouter` still charges 0.5% — read back live the same day, `suitsNft()` is the zero address, `feeBps()` 50, `holderFeeBps()` 0 — so the badge now also requires the router's own `suitsNft` to be set and matching, and turns itself on when `setSuitsNft` is called (D34). Non-holders are not redirected: they are told which wallet was checked and offered the collection at `opensea.io/collection/the-suits-nft`. Amended Sep 28, 2026 (K): token pages are no longer gated, see D46. Superseded Oct 9, 2026 (K): the gate is removed outright, see D47. | K, C |
| D45 | Product name is MaxLong Markets | Sep 28, 2026 (K). The product is **MaxLong Markets**, superseding D10's LongMarkets. It had been carrying three names at once — LongMarkets in this file, "SuitVentures Markets" in the page title and the wallet's sign-in message, and maxlong.markets as the domain — which split the brand across three strings and gave search engines three things to rank instead of one. SuitVentures stays as the studio that builds it, and keeps the SUIT token and the Suits NFT; only the product is renamed. Changed everywhere it is user-visible: the page title and its template, applicationName, the wagmi and RainbowKit app names, the SIWE statement, the Yellowtail wordmark in the top bar and on the lock screen, and the logo fetcher's User-Agent. Nobody is signed out by the new SIWE statement: it is display-only, `verifySiwe` never compares it, and the session cookie is an HMAC over the address and its timestamps. The proposal and article titles in section 1's Sources, and the dated Decision Log rows that name the old product, are left verbatim because they record what was written then. | K, C |
| D46 | Token pages are public and indexable | Sep 28, 2026 (K). `/t/<address>` comes back out from behind the gate and serves a server-rendered facts layer any visitor and any crawler can read: ticker, name, address, quote asset, origin, price, 1h and 24h change, 24h volume, market cap, FDV, liquidity, holder count, age and fee tier, plus per-token metadata and JSON-LD. Everything interactive — the chart, the trade list, top holders and Buy/Sell — stays behind the same Suits check, in a compact locked card rather than the full lock screen. This amends D44's "every route": the markets table, traders and watchlist are still gated, and the sitemap is what makes the token pages discoverable at all, since a gated `/` leaves no crawl path to them. Serving crawlers something the visitor does not get was rejected outright: that is cloaking, and it risks the whole domain. Exposure is unchanged in substance — the same facts already come back unauthenticated from `/api/tokens/<address>` and the public pairs route, so this publishes nothing the front door was keeping. Canonical host is `www.maxlong.markets`; the apex redirects to it. Amended Oct 9, 2026 (K): with the gate gone (D47) the whole app is public, and the facts block stays in the HTML for crawlers but off screen for everyone, since the details card under the trade panel shows the same figures. | K, C |
| D47 | Public release: the Suits gate is removed | Oct 9, 2026 (K). The holders-only gate of D44 comes down and the whole app is open to the public: the markets table, token pages, watchlist and launch page render for any visitor, wallet or no wallet, and nothing checks a Suits balance for access any more. This is a code removal, not the `NEXT_PUBLIC_TOKEN_GATE=off` switch D44 reserved: the lock screen, the locked card on token pages, the gate wrapper and the access hook are deleted, and the flag is gone from the config. Section 1's "browsing is open to anyone" is back in force. What stays: the Suits NFT address remains configured for the router fee perk alone (D34), and the fee badge keeps its rule of requiring the router's own `suitsNft` to match. The sign-in ask at connect no longer waits on a Suits read. The launch page joins the sitemap as a real public page; the watchlist stays out, since a crawler holding no wallet gets only the signed-out empty copy. Not part of this decision: `SVLaunchpad`'s on-chain `launcherGate` still restricts *creating* a token to Suits holders until its owner calls `setLauncherGate(0)`, and the launch form still says so for a wallet without one. | K, C |

---

## 8. Open items

Nothing here blocks starting M1. Each has an owner and a deadline.

| Item | Owner | Needed by |
|---|---|---|
| **Overdue.** Pons aggregation is scope added Sep 10, after the fixed price was agreed. Decide whether it is absorbed into the fixed price or billed as a change order at the day rate, and record the outcome in the Decision Log. | Kobe / MAXLONG | Day 2 (passed) |
| SUIT / PLTR pool key, confirm no hook | MAXLONG | M1 |
| Kobe's EOA address recorded in Environments | Kobe | Day 2 |
| Verify D30: register one known token, sync, and compare the derived `totalSupply` against `totalSupply()` over RPC. | Kobe | Day 2 |
| Confirm with Envio support whether the 100k and 1M plan limits count stored or processed events. The indexer subscribes to PoolManager chain-wide and filters in the handler, so a processed-event limit would be hit far sooner than the section 4 estimate assumes. | Kobe | Before paying for a plan |
| Confirm the exact Suits NFT holder fee discount with the client (0% proposed, D34). | Kobe / MAXLONG | Before M2 starts |
| USD fallback for the 26 allowlisted stock tokens without a Chainlink feed (DJT, RDDT, TTWO, GLD and others; 702 of the 3,647 allowlisted Pons pools in the snapshot). M1 promises live USD prices for every row. Options: show the quote-asset price with USD as a dash, or derive USD from a USDG pool. | Kobe / MAXLONG | M1 |
| Verify Robinhood Stock Token contracts carry no transfer allowlist or blocklist (SVRouter holds the input for the length of a swap). **Resolved for NVDA on Sep 17, 2026:** SVRouter's fork suite moves NVDA through Permit2, the router and the Universal Router on a mainnet fork; the other stock tokens share the issuer's contract and are assumed alike until the live test. | Kobe | M2 |
| Confirm Safe is deployed on chain 4663, else propose fallback | Kobe | Day 2 |
| Collect and record all TBD addresses in section 2 | Kobe | M1 |
| Article wording: the article describes a launch layer (factory, presales, Create) that is out of scope under D31; "SuitVentures indexes LONG" is now every launchpad's v4 pools; "one-tap swap" is tap plus wallet confirm under Option B; "stock tokens are the only quote assets" is now stock tokens plus USDG (D22). The name LongMarkets also reads oddly for a product that aggregates every launchpad. | MAXLONG | Before public launch |
| ~~**Disk size for the market store.**~~ **Closed Sep 21, 2026 (K).** The windows in D39 as amended settle the question without buying disk. Trades were pruned to 5 days (14,436,843 rows deleted, 2.2 M left), the by-pool-and-time index on trades was rebuilt, and a full vacuum took the trades table from 7,625 MB to 961 MB and the whole database from 8,831 MB to 2,167 MB. At these windows the store settles near 3 GB and stops growing, so no resize is needed. The Envio copy this item was blocking is moot: the hosted deployment was cancelled and the reader now folds the chain's own events (D29). | Kobe | Closed |
| **Traders board switched off on production.** The 24-hour ranking takes about 40 seconds on the copied trades because they were written in id order, not time order, so the timestamp index prunes nothing. Fix is a one-time rewrite of the trades table in time order (needs about twice the table in free disk, so it rides on the disk decision above) or an hourly rollup written by the sync (M5). Until then `TRADERS_BOARD=off` keeps `/traders` on its "needs the market store" message. **Update Sep 21, 2026 (K):** the disk dependency is gone and the rewrite has happened — the prune to 5 days, the rebuilt by-pool-and-time index and a full vacuum took a token's trades query from 38 s to 0.65 s on 2.2 M rows. The board's own 24-hour ranking has not been re-timed yet; test it after the reader is live and turn the switch on if it answers quickly. | Kobe | After cutover |

---

## 9. Out of scope for v1

From the brief: vesting, LongLauncher as minter, LongX, native mobile app, gasless, Apple Pay, multi-chain, Community Mode hooks, docs page, terminal UI, vesting UI.

Added: token creation or launching of any kind, presales, trading of tokens still on a bonding curve (curves that run inside a Uniswap v4 pool are listed and linked out to their launchpad, not routed; Pons curves are separate contracts and are not listed), limit orders, price alerts, comments or chat, third-party security audit, automated fee allocation (buyback, NFT sweeps, PLTR vault), fee dashboard, embedded wallets (Privy), pool-level fee hook, self-updating quote list, Pons v1 tokens in Uniswap v3 pools, tokens quoted in non-allowlisted assets (ETH, WETH, AI, cbBTC and others, D22).

Separate engagements, quoted separately: The Suits token deployment, airdrop claim page (30% of SUIT to Suits NFT holders).

Work outside this scope: USD 350 per day, agreed in writing first.

---

## 10. Risks

- **No audit.** SVRouter is not third-party audited. It holds no funds between transactions and its logic is small; fork tests against live Uniswap v4 and small live trades are the mitigation. Shipping unaudited is the client's recorded decision.
- **No testnet for LONG.** Mainnet fork, then live with small amounts.
- **LONG or Pons changes.** If either platform changes its hook, launcher, or token behavior in a way that blocks routing or indexing, the schedule pauses until a path is agreed.
- **Envio pause freezes prices until it is back.** The app reads its own store (D39), so a paused, re-syncing or replaced indexer costs freshness, not the table: prices, volumes and candles stop moving until the sync resumes, and `/api/health` shows the sync age. Rebuilding the indexer behind the sync is invisible to visitors.
- **Pons hook parameters are owner-settable.** `hookFeeBps` and the creator tax cap can change by owner transaction. The quote line reads the pool fee live rather than hardcoding 1%.
- **Fee leakage.** Router-only fee. Other frontends trade the same pools for free.
- **Thin pools.** A small swap can move price materially.
- **Stock token characteristics.** Issuance, redemption, and market hours affect quote-side pricing independently of the community token.
- **Non-custodial.** MaxLong Markets cannot pause tokens and cannot reverse transactions.

Nothing in this file is investment advice.

---

## 11. Working agreements

- Written progress update Monday and Thursday: shipped, next, waiting on.
- Each milestone ends with a short demo. Acceptance or specific change requests within 48 hours; silence counts as acceptance.
- Replies to open questions within 24 hours keep the schedule; the 24-day clock pauses for longer waits and for LONG-side blockers.
- Kobe deploys and holds admin during the build, then transfers contract ownership and repository access at handover. General tooling and non-client-specific code may be reused.

## Repositories

This folder is the workspace repo (`KobeSergio/suitventures-markets`): it holds this README (the Decision Log is the source of truth), `research/`, and the app folders as git submodules. Each app keeps its own repo and its own CI/CD (Vercel for the frontend, Envio Cloud for the indexer).

| Folder | Repo |
|---|---|
| `SuitVenturesMarket-frontend/` | https://github.com/KobeSergio/suitventures-markets-frontend |
| `SuitVenturesMarket-indexer/` | https://github.com/KobeSergio/suitventures-markets-indexer |

Clone with `git clone --recurse-submodules`. After pushing new commits in an app repo, run `git add <folder> && git commit` here to move the submodule pointer.
